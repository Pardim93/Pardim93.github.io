---
layout: post
author: Wellington Pardim
title:  "Pip e Virtualenv: Configurando seu ambiente Python"
categories: "python productivity"
position: 7
---

Um dos primeiros problemas que todo desenvolvedor Python enfrenta é o gerenciamento de dependências. Instalar pacotes globalmente funciona no início, mas rapidamente vira uma bagunça quando projetos diferentes exigem versões diferentes da mesma biblioteca. A solução são ambientes virtuais.

## O problema

Imagine que você tem dois projetos:

- Projeto A precisa de `requests==2.25.0`
- Projeto B precisa de `requests==2.28.0`

Se instalar globalmente, um vai quebrar o outro. Com virtualenv, cada projeto tem seu próprio ambiente isolado com suas próprias dependências.

## Pip: o gerenciador de pacotes

O `pip` é a ferramenta padrão para instalar pacotes Python do [PyPI](https://pypi.org). Comandos essenciais:

```bash
# Instalar um pacote
pip install requests

# Instalar versão específica
pip install requests==2.25.0

# Listar pacotes instalados
pip list

# Gerar arquivo com todas as dependências
pip freeze > requirements.txt

# Instalar dependências de um arquivo
pip install -r requirements.txt

# Atualizar um pacote
pip install --upgrade requests

# Desinstalar
pip uninstall requests
```

O arquivo `requirements.txt` é a forma mais comum de declarar dependências:

```txt
flask==2.0.1
requests>=2.25,<3.0
python-dotenv==0.19.0
```

## Venv: ambientes virtuais nativos

Desde o Python 3.3, o módulo `venv` faz parte da biblioteca padrão. Não precisa instalar nada:

```bash
# Criar um ambiente virtual
python3 -m venv venv

# Ativar no Linux/macOS
source venv/bin/activate

# Ativar no Windows
venv\Scripts\activate

# Desativar
deactivate
```

Quando o ambiente está ativado, o prompt do terminal muda para indicar qual ambiente está ativo:

```bash
(venv) $ python --version
Python 3.9.7
(venv) $ pip install flask
(venv) $ pip freeze > requirements.txt
```

O `pip` dentro do ambiente instala pacotes apenas nele, sem afetar o sistema.

## Estrutura de projeto recomendada

```
meu-projeto/
├── venv/               # ambiente virtual (não commitar!)
├── src/                # código fonte
│   └── app.py
├── requirements.txt    # dependências
├── .gitignore          # excluir venv/, __pycache__, etc.
└── README.md
```

O `.gitignore` deve conter:

```
venv/
__pycache__/
*.pyc
.env
*.egg-info/
dist/
build/
```

**Nunca commit o diretório `venv/`.** O `requirements.txt` é suficiente para recriar o ambiente.

## Virtualenv vs venv vs virtualenvwrapper

Existem várias ferramentas para ambientes virtuais:

**venv** (recomendado para iniciantes):
- Já vem com o Python 3
- Simples e direto
- Sem instalação adicional

**virtualenv**:
- Mais antigo, funciona com Python 2 e 3
- Um pouco mais rápido na criação
- `pip install virtualenv`

**virtualenvwrapper**:
- Gerencia múltiplos ambientes em um diretório central
- Comandos como `mkvirtualenv`, `workon`, `rmvirtualenv`
- Útil quando você trabalha em muitos projetos simultaneamente

```bash
pip install virtualenvwrapper
export WORKON_HOME=~/envs
source /usr/local/bin/virtualenvwrapper.sh

# Criar e ativar
mkvirtualenv meu-projeto
workon meu-projeto
```

## Dicas práticas

**1. Sempre use virtualenv para cada projeto novo:**

```bash
mkdir novo-projeto && cd novo-projeto
python3 -m venv venv
source venv/bin/activate
pip install flask
pip freeze > requirements.txt
```

**2. Atualize o pip dentro do ambiente:**

```bash
pip install --upgrade pip
```

**3. Use `pip-tools` para dependências mais organizadas:**

```bash
pip install pip-tools
```

Crie um `requirements.in` com as dependências principais:

```txt
flask
requests
```

Gere o `requirements.txt` com versões travadas:

```bash
pip-compile requirements.in
```

**4. Considere o `pyproject.toml`** — o formato moderno de configuração de projetos Python está migrando para `pyproject.toml` em vez de `setup.py`. Ferramentas como `poetry` e `pipenv` já o utilizam.

## Conclusão

Gerenciar ambientes virtuais é uma prática fundamental para qualquer desenvolvedor Python. Comece com `venv` (que já vem com o Python) e `requirements.txt`. Conforme seus projetos crescerem, explore ferramentas como `pip-tools` ou `poetry` para um controle mais sofisticado.

O importante é: **um ambiente virtual por projeto, sempre.**

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
