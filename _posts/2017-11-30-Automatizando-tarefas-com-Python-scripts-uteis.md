---
layout: post
author: Wellington Pardim
title:  "Automatizando tarefas com Python: scripts úteis no dia a dia"
categories: "python productivity"
position: 11
---

Uma das maiores vantagens do Python é a facilidade de automatizar tarefas repetitivas. Nesse artigo vou compartilhar 5 scripts que uso no dia a dia e que você pode copiar e adaptar para as suas necessidades.

## 1. Renomeador de arquivos em lote

Precisa renomear centenas de arquivos com um padrão? Esse script faz isso em segundos:

```python
from pathlib import Path

def renomear_arquivos(diretorio, prefixo, extensao=None):
    """
    Renomeia arquivos em um diretório adicionando um prefixo sequencial.
    
    Uso: renomear_arquivos("/fotos", "ferias", ".jpg")
    Resultado: ferias_001.jpg, ferias_002.jpg, ...
    """
    caminho = Path(diretorio)
    arquivos = sorted(caminho.glob(f"*{extensao}" if extensao else "*"))
    
    for i, arquivo in enumerate(arquivos, 1):
        if arquivo.is_file():
            novo_nome = caminho / f"{prefixo}_{i:03d}{arquivo.suffix}"
            arquivo.rename(novo_nome)
            print(f"{arquivo.name} → {novo_nome.name}")

# Exemplo de uso
renomear_arquivos("/home/user/fotos", "ferias", ".jpg")
```

O `pathlib` é a forma moderna de manipular caminhos no Python. Substitui boa parte do que fazíamos com `os.path`.

## 2. Web scraper simples

Para extrair dados de páginas web:

```python
import requests
from bs4 import BeautifulSoup

def raspar_noticias(url):
    """
    Extrai títulos e links de uma página de notícias.
    """
    resposta = requests.get(url, timeout=10)
    resposta.raise_for_status()
    
    soup = BeautifulSoup(resposta.text, "html.parser")
    
    resultados = []
    for link in soup.find_all("a", href=True):
        titulo = link.get_text(strip=True)
        href = link["href"]
        if titulo and len(titulo) > 20:  # filtra links curtos
            resultados.append({"titulo": titulo, "link": href})
    
    return resultados

# Exemplo de uso
noticias = raspar_noticias("https://news.ycombinator.com")
for n in noticias[:10]:
    print(f"{n['titulo']}")
    print(f"  {n['link']}\n")
```

Dependências:

```bash
pip install requests beautifulsoup4
```

## 3. Processador de CSV

Transforme e filtre dados de CSV facilmente:

```python
import csv
from pathlib import Path

def filtrar_csv(entrada, saida, coluna, valor):
    """
    Filtra linhas de um CSV onde uma coluna tem um valor específico.
    
    Uso: filtrar_csv("vendas.csv", "vendas_sp.csv", "estado", "SP")
    """
    with open(entrada, newline="", encoding="utf-8") as arq_entrada:
        leitor = csv.DictReader(arq_entrada)
        
        with open(saida, "w", newline="", encoding="utf-8") as arq_saida:
            campos = leitor.fieldnames
            escritor = csv.DictWriter(arq_saida, fieldnames=campos)
            escritor.writeheader()
            
            count = 0
            for linha in leitor:
                if linha[coluna] == valor:
                    escritor.writerow(linha)
                    count += 1
    
    print(f"{count} linhas filtradas salvas em {saida}")

def resumir_csv(entrada, coluna_grupo, coluna_valor):
    """
    Gera um resumo agrupando por uma coluna e somando outra.
    
    Uso: resumir_csv("vendas.csv", "vendedor", "valor")
    """
    from collections import defaultdict
    
    totais = defaultdict(float)
    
    with open(entrada, newline="", encoding="utf-8") as arq:
        for linha in csv.DictReader(arq):
            totais[linha[coluna_grupo]] += float(linha[coluna_valor])
    
    for grupo, total in sorted(totais.items(), key=lambda x: -x[1]):
        print(f"{grupo}: R$ {total:,.2f}")

# Exemplos de uso
filtrar_csv("vendas.csv", "vendas_sp.csv", "estado", "SP")
resumir_csv("vendas.csv", "vendedor", "valor")
```

## 4. Enviador de e-mails automatizado

Para enviar e-mails em lote (relatórios, notificações):

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def enviar_email(destinatario, assunto, corpo, remetente, senha):
    """
    Envia um e-mail usando SMTP do Gmail.
    
    Para usar com Gmail, ative "App Passwords" nas configurações de segurança.
    """
    msg = MIMEMultipart()
    msg["From"] = remetente
    msg["To"] = destinatario
    msg["Subject"] = assunto
    
    msg.attach(MIMEText(corpo, "html"))
    
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as servidor:
        servidor.login(remetente, senha)
        servidor.send_message(msg)
    
    print(f"E-mail enviado para {destinatario}")

def enviar_em_lote(destinatarios, assunto, corpo, remetente, senha):
    """
    Envia o mesmo e-mail para uma lista de destinatários.
    """
    for dest in destinatarios:
        try:
            enviar_email(dest, assunto, corpo, remetente, senha)
        except Exception as e:
            print(f"Erro ao enviar para {dest}: {e}")

# Exemplo de uso
enviar_email(
    destinatario="destino@email.com",
    assunto="Relatório semanal",
    corpo="<h1>Relatório</h1><p>Segue em anexo.</p>",
    remetente="seu@email.com",
    senha="sua-app-password"
)
```

**Importante**: nunca coloque senhas diretamente no código. Use variáveis de ambiente:

```python
import os
senha = os.environ.get("EMAIL_PASSWORD")
```

## 5. Agendador de tarefas

Execute tarefas em intervalos regulares sem precisar de cron:

```python
import schedule
import time

def tarefa_backup():
    """Executa backup do banco de dados."""
    from datetime import datetime
    print(f"[{datetime.now()}] Iniciando backup...")
    # Aqui você colocaria seu comando de backup
    # subprocess.run(["pg_dump", "meubanco", "-f", "backup.sql"])

def tarefa_limpeza():
    """Remove arquivos temporários mais antigos que 7 dias."""
    from pathlib import Path
    from datetime import datetime, timedelta
    
    temp = Path("/tmp/meuapp")
    limite = datetime.now() - timedelta(days=7)
    
    for arquivo in temp.glob("*"):
        if arquivo.is_file():
            modificado = datetime.fromtimestamp(arquivo.stat().st_mtime)
            if modificado < limite:
                arquivo.unlink()
                print(f"Removido: {arquivo.name}")

# Agendamento
schedule.every().day.at("02:00").do(tarefa_backup)
schedule.every(6).hours.do(tarefa_limpeza)

print("Agendador rodando. Ctrl+C para parar.")
while True:
    schedule.run_pending()
    time.sleep(60)
```

Dependência:

```bash
pip install schedule
```

## Dicas gerais

**Use `argparse` para scripts reutilizáveis** — transforme seus scripts em ferramentas de linha de comando:

```python
import argparse

parser = argparse.ArgumentParser(description="Processa CSV")
parser.add_argument("entrada", help="Arquivo CSV de entrada")
parser.add_argument("-f", "--filtro", help="Filtrar por coluna=valor")

args = parser.parse_args()
```

**Trate erros com `try/except`** — scripts automatizados devem ser resilientes:

```python
try:
    resultado = requests.get(url, timeout=10)
    resultado.raise_for_status()
except requests.exceptions.Timeout:
    print("Timeout - tentando novamente...")
except requests.exceptions.HTTPError as e:
    print(f"Erro HTTP: {e}")
```

**Use logging em vez de print** — para scripts que rodam em background:

```python
import logging

logging.basicConfig(
    filename="automacao.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("Script iniciado")
logging.error("Erro ao processar arquivo")
```

## Conclusão

Python é a linguagem ideal para automação. Com poucas linhas de código, você pode economizar horas de trabalho manual. Os scripts acima são apenas pontos de partida — adapte-os para as suas necessidades específicas.

Comece identificando tarefas repetitivas no seu dia a dia. Se você faz algo mais de uma vez por semana, provavelmente pode automatizar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
