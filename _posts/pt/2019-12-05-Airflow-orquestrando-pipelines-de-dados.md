---
layout: post
author: Wellington Pardim
title:  "Apache Airflow: orquestrando pipelines de dados com Python"
categories: "data-engineering python"
position: 19
---

Se você trabalha com dados, sabe que a parte difícil não é extrair ou transformar — é orquestrar tudo para rodar na ordem certa, no horário certo, com tratamento de erros. É aí que entra o **Apache Airflow**.

## O que é Airflow?

Airflow é uma plataforma open-source para criar, agendar e monitorar workflows (chamados de **DAGs** — Directed Acyclic Graphs). Cada etapa do workflow é uma **task**, e as tasks são definidas em Python.

Vantagens:

- **Workflows como código** — versionável, testável, reutilizável
- **Interface web** para monitorar execuções
- **Agendamento flexível** — cron expressions, intervalos, sensores
- **Extensível** — operadores para praticamente qualquer serviço

## Instalação

A forma mais rápida é com Docker:

```bash
pip install apache-airflow
```

Ou com Docker Compose:

```yaml
# docker-compose.yml
version: "3.8"
services:
  airflow-webserver:
    image: apache/airflow:2.5.0
    ports:
      - "8080:8080"
    volumes:
      - ./dags:/opt/airflow/dags
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.5.0
    volumes:
      - ./dags:/opt/airflow/dags
    command: scheduler
```

## Conceitos fundamentais

- **DAG**: define as dependências entre tasks (o "mapa" do workflow)
- **Task**: uma unidade de trabalho (extrair dados, transformar, enviar e-mail)
- **Operator**: o tipo de task (PythonOperator, BashOperator, etc.)
- **Sensor**: task que espera uma condição (arquivo existir, API responder)
- **Executor**: como as tasks são executadas (local, Celery, Kubernetes)

## DAG completa: ETL simples

Vamos construir um pipeline que extrai dados de uma API, transforma e salva em CSV:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

# Configurações padrão para todas as tasks
default_args = {
    "owner": "wellington",
    "depends_on_past": False,
    "email_on_failure": False,
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
}

# Definição do DAG
dag = DAG(
    "pipeline_vendas",
    default_args=default_args,
    description="Pipeline de vendas diárias",
    schedule_interval="0 6 * * *",  # Todo dia às 6h
    start_date=datetime(2019, 1, 1),
    catchup=False,  # Não executar DAGs passados
    tags=["vendas", "etl"],
)

def extrair_dados(**context):
    """Extrai dados da API de vendas."""
    import requests
    import json

    ontem = context["ds"]  # Data de execução (ds = date string)
    url = f"https://api.exemplo.com/vendas?data={ontem}"

    resposta = requests.get(url, timeout=30)
    resposta.raise_for_status()

    dados = resposta.json()
    print(f"Extraídas {len(dados)} vendas")

    # Salva para próxima task usar
    with open("/tmp/vendas_raw.json", "w") as f:
        json.dump(dados, f)

    return len(dados)

def transformar_dados(**context):
    """Transforma e limpa os dados."""
    import json
    import csv

    with open("/tmp/vendas_raw.json", "r") as f:
        dados = json.load(f)

    # Filtrar vendas válidas
    vendas_validas = [
        v for v in dados
        if v.get("valor", 0) > 0 and v.get("status") == "concluida"
    ]

    # Calcular totais
    total = sum(v["valor"] for v in vendas_validas)
    media = total / len(vendas_validas) if vendas_validas else 0

    print(f"Vendas válidas: {len(vendas_validas)}")
    print(f"Total: R$ {total:,.2f}")
    print(f"Média: R$ {media:,.2f}")

    # Salvar CSV
    with open("/tmp/vendas_processadas.csv", "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=["id", "valor", "data", "cliente"])
        writer.writeheader()
        for v in vendas_validas:
            writer.writerow({
                "id": v["id"],
                "valor": v["valor"],
                "data": v["data"],
                "cliente": v["cliente"]["nome"],
            })

    # Passar dados para próxima task via XCom
    context["task_instance"].xcom_push(key="total_vendas", value=total)
    context["task_instance"].xcom_push(key="qtd_vendas", value=len(vendas_validas))

def enviar_relatorio(**context):
    """Envia relatório por e-mail."""
    ti = context["task_instance"]
    total = ti.xcom_pull(key="total_vendas", task_ids="transformar")
    qtd = ti.xcom_pull(key="qtd_vendas", task_ids="transformar")
    data = context["ds"]

    print(f"Relatório {data}:")
    print(f"  Vendas: {qtd}")
    print(f"  Total: R$ {total:,.2f}")
    # Aqui você integraria com smtplib ou SendGrid

# Definição das tasks
t1 = PythonOperator(
    task_id="extrair",
    python_callable=extrair_dados,
    dag=dag,
)

t2 = PythonOperator(
    task_id="transformar",
    python_callable=transformar_dados,
    dag=dag,
)

t3 = PythonOperator(
    task_id="relatorio",
    python_callable=enviar_relatorio,
    dag=dag,
)

t4 = BashOperator(
    task_id="limpeza",
    bash_command="rm -f /tmp/vendas_raw.json",
    dag=dag,
)

# Dependências: t1 → t2 → t3 → t4
t1 >> t2 >> t3 >> t4
```

## XCom: comunicação entre tasks

XCom (Cross-Communication) permite que tasks compartilhem dados:

```python
# Push: enviar dados
def task_a(**context):
    context["task_instance"].xcom_push(key="resultado", value=42)

# Pull: receber dados
def task_b(**context):
    valor = context["task_instance"].xcom_pull(
        key="resultado", task_ids="task_a"
    )
    print(f"Recebeu: {valor}")  # 42
```

**Cuidado**: XCom é para dados pequenos (metadados, contadores). Para dados grandes, use arquivos ou banco de dados.

## Sensors: esperando condições

```python
from airflow.sensors.filesystem import FileSensor

esperar_arquivo = FileSensor(
    task_id="esperar_arquivo",
    filepath="/data/vendas_{{ ds }}.csv",
    poke_interval=30,  # Verifica a cada 30 segundos
    timeout=3600,      # Timeout após 1 hora
    dag=dag,
)

esperar_arquivo >> t1
```

## Interface web

O Airflow tem uma interface web poderosa em `localhost:8080`:

- **DAGs**: lista de todos os workflows com status
- **Tree View**: visualização das execuções ao longo do tempo
- **Graph View**: grafo visual das dependências
- **Logs**: logs detalhados de cada task
- **Gantt**: timeline de execução

## Conclusão

Apache Airflow é a ferramenta padrão da indústria para orquestração de pipelines de dados. Definir workflows em Python permite versionamento, testes e reutilização que ferramentas visuais não oferecem.

Para começar, pense em uma tarefa repetitiva no seu dia a dia — um relatório que roda toda segunda, uma sincronização de dados, um backup — e transforme isso em uma DAG.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
