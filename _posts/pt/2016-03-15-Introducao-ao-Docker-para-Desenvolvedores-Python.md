---
layout: post
author: Wellington Pardim
title:  "Introdução ao Docker para Desenvolvedores Python"
categories: "docker python"
position: 4
---

Docker tem se tornado uma ferramenta essencial no dia a dia de desenvolvedores. Se você trabalha com Python, saber containerizar suas aplicações pode simplificar muito o deploy e garantir que o ambiente de produção seja idêntico ao de desenvolvimento.

## O que é Docker?

Docker é uma plataforma que permite empacotar aplicações e suas dependências em **containers**. Um container é uma unidade leve e portátil que roda isolada do sistema operacional, diferente de uma máquina virtual que precisa de um SO inteiro.

A grande vantagem para desenvolvedores Python é acabar com o clássico "na minha máquina funciona". Se roda no container, roda em qualquer lugar.

## Instalando o Docker

No Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install docker-ce
```

No macOS, baixe o [Docker Desktop](https://www.docker.com/products/docker-desktop). Após a instalação, verifique se tudo funcionou:

```bash
docker --version
```

## Criando um Dockerfile para Python

O `Dockerfile` é o "receita" do seu container. Veja um exemplo para uma aplicação Flask simples:

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Olá mundo com Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

```txt
# requirements.txt
flask==2.0.1
```

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Cada instrução faz o seguinte:

- **FROM**: define a imagem base (Python 3.9 slim para manter o container leve)
- **WORKDIR**: define o diretório de trabalho dentro do container
- **COPY**: copia arquivos do seu projeto para o container
- **RUN**: executa comandos durante a build (instalar dependências)
- **EXPOSE**: documenta qual porta a aplicação usa
- **CMD**: comando que roda quando o container inicia

## Build e Run

Para construir a imagem:

```bash
docker build -t meu-app-python .
```

Para rodar o container:

```bash
docker run -p 5000:5000 meu-app-python
```

Agora acesse `http://localhost:5000` no navegador. A aplicação está rodando dentro de um container.

## Docker Compose para múltiplos serviços

Quando sua aplicação depende de outros serviços (banco de dados, cache, etc), o Docker Compose facilita a orquestração. Veja um exemplo com Python e Redis:

```yaml
# docker-compose.yml
version: "3.8"

services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

Para subir tudo de uma vez:

```bash
docker-compose up
```

## Dicas para Python com Docker

Use um `.dockerignore` para evitar copiar arquivos desnecessários:

```
__pycache__
*.pyc
.git
.env
venv/
```

Uma dica importante: copie o `requirements.txt` **antes** do resto do código. O Docker cacheia as camadas, então se apenas o código mudar (e não as dependências), o `pip install` não será executado novamente, acelerando muito os builds.

## Conclusão

Docker resolve um dos maiores dores de cabeça do desenvolvimento: a inconsistência entre ambientes. Para desenvolvedores Python, é uma ferramenta que vale muito a pena aprender. Comece containerizando projetos pequenos e em breve não vai mais querer trabalhar sem.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
