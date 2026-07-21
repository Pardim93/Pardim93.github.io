---
layout: post
author: Wellington Pardim
title:  "dbt: transformando dados com SQL e Python"
categories: "data-engineering"
position: 27
---

Se você trabalha com dados, provavelmente passa mais tempo **transformando** do que extraindo. O **dbt** (Data Build Tool) é a ferramenta que trouxe práticas de engenharia de software para o mundo de transformação de dados: versionamento, testes, documentação e modularity — tudo em SQL.

## O que é dbt?

dbt é uma ferramenta open-source que permite transformar dados no warehouse usando **SQL SELECT statements**. Em vez de escrever scripts ETL complexos, você escreve queries SQL e o dbt cuida de:

- Materializar como tabelas ou views
- Gerenciar dependências entre models
- Rodar testes de qualidade
- Gerar documentação automática

O fluxo é: **Extract & Load** (ferramentas como Airbyte, Fivetran) → **Transform** (dbt).

## Instalação

```bash
pip install dbt-core
# ou, para PostgreSQL:
pip install dbt-postgres
# para BigQuery:
pip install dbt-bigquery
```

## Inicializando um projeto

```bash
dbt init meu_projeto
cd meu_projeto
```

Estrutura criada:

```
meu_projeto/
├── dbt_project.yml    # Configuração do projeto
├── models/            # Queries SQL
│   ├── staging/       # Models de staging (limpeza)
│   └── marts/         # Models finais (analytics)
├── tests/             # Testes customizados
├── macros/            # Funções SQL reutilizáveis
└── seeds/             # CSVs para carregar dados estáticos
```

## Escrevendo models

Cada model é um arquivo `.sql` com um SELECT. O dbt materializa como view ou tabela.

### Staging: limpando dados brutos

```sql
-- models/staging/stg_pedidos.sql

with source as (
    select * from {{ source('raw', 'pedidos') }}
),

cleaned as (
    select
        id,
        cliente_id,
        valor,
        status,
        created_at::date as data_pedido,
        case
            when status = 'completed' then 'concluido'
            when status = 'pending' then 'pendente'
            when status = 'cancelled' then 'cancelado'
            else 'desconhecido'
        end as status_traduzido
    from source
    where valor > 0
      and created_at >= '2020-01-01'
)

select * from cleaned
```

O `{{ source('raw', 'pedidos') }}` é uma **Jinja** template — dbt injeta o nome correto da tabela.

### Mart: dados prontos para análise

```sql
-- models/marts/fct_vendas.sql

with pedidos as (
    select * from {{ ref('stg_pedidos') }}
),

clientes as (
    select * from {{ ref('stg_clientes') }}
),

vendas as (
    select
        p.id as pedido_id,
        p.data_pedido,
        c.nome as cliente_nome,
        c.estado as cliente_estado,
        p.valor,
        p.status_traduzido
    from pedidos p
    inner join clientes c on p.cliente_id = c.id
    where p.status_traduzido = 'concluido'
)

select * from vendas
```

O `{{ ref('stg_pedidos') }}` cria uma dependência — dbt sabe que `stg_pedidos` deve rodar antes de `fct_vendas`.

## Materializações

Controla como o model é persistido no warehouse:

```yaml
# dbt_project.yml
models:
  meu_projeto:
    staging:
      +materialized: view
    marts:
      +materialized: table
```

Opções:

| Tipo | Quando usar |
|------|------------|
| `view` | Staging, dados que mudam frequentemente |
| `table` | Marts, dados que são caros de calcular |
| `incremental` | Grandes volumes, append de dados novos |

### Incremental models

Para tabelas grandes, processe apenas dados novos:

```sql
-- models/marts/fct_eventos.sql
{{
    config(
        materialized='incremental',
        unique_key='evento_id'
    )
}}

select
    evento_id,
    usuario_id,
    tipo,
    timestamp
from {{ ref('stg_eventos') }}

{% if is_incremental() %}
  where timestamp > (select max(timestamp) from {{ this }})
{% endif %}
```

## Testes

dbt vem com testes genéricos e suporta testes customizados:

```yaml
# models/staging/stg_pedidos.yml
version: 2

models:
  - name: stg_pedidos
    columns:
      - name: id
        tests:
          - unique
          - not_null

      - name: valor
        tests:
          - not_null
          - dbt_utils.accepted_range:
              min_value: 0
              max_value: 1000000

      - name: status_traduzido
        tests:
          - accepted_values:
              values: ['concluido', 'pendente', 'cancelado']
```

Execute os testes:

```bash
dbt test
```

## Documentação automática

Gere docs interativas com um comando:

```bash
dbt docs generate
dbt docs serve
```

Isso abre uma interface web em `localhost:8080` com:
- Grafo de dependências entre models
- Descrição de cada coluna
- Testes associados
- SQL de cada model

## Macros: SQL reutilizável

```sql
-- macros/formata_moeda.sql
{% macro formata_moeda(coluna) %}
    'R$ ' || to_char({{ coluna }}, 'FM999G999G999D00')
{% endmacro %}

-- Uso em um model
select
    pedido_id,
    {{ formata_moeda('valor') }} as valor_formatado
from {{ ref('stg_pedidos') }}
```

## Comandos essenciais

```bash
# Rodar todos os models
dbt run

# Rodar um model específico
dbt run --select stg_pedidos

# Rodar um model e seus downstream
dbt run --select fct_vendas+

# Rodar testes
dbt test

# Seed (carregar CSVs)
dbt seed

# Snapshot (Type 2 SCD)
dbt snapshot

# Documentação
dbt docs generate
dbt docs serve

# Debug de conexão
dbt debug
```

## Conclusão

dbt mudou a forma como pensamos sobre transformação de dados. Em vez de scripts ETL frágeis, temos SQL versionado, testado e documentado. A curva de aprendizado é baixa — se você sabe SQL, já sabe usar dbt.

Comece com um projeto simples: crie 2-3 staging models e 1 mart. Adicione testes e documentação. Em poucas horas, terá um pipeline de dados profissional.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
