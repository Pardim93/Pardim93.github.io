---
layout: post
author: Wellington Pardim
title:  "Python 3.5: O que mudou e por que migrar"
categories: "python"
position: 5
---

Em 2015 o Python 3.5 foi lançado e trouxe mudanças significativas que tornaram a migração do Python 2 ainda mais justificável. Se você ainda está rodando Python 2.7 (ou até 3.3/3.4), vale a pena conhecer o que há de novo.

## Por que migrar?

Python 2.7 chegou ao fim de vida em janeiro de 2020. Isso significa que não há mais atualizações de segurança, correções de bugs ou suporte oficial. Muitas bibliotecas já abandonaram o suporte ao Python 2. Se você ainda não migrou, agora é a hora.

## Async/Await nativo

Essa é talvez a feature mais esperada. O Python 3.5引入了语法级别的 suporte a programação assíncrona com as palavras-chave `async` e `await`:

```python
import asyncio

async def buscar_dados(url):
    print(f"Buscando dados de {url}...")
    await asyncio.sleep(2)  # simula uma operação de I/O
    return {"dados": "exemplo"}

async def main():
    resultado = await buscar_dados("https://api.exemplo.com")
    print(resultado)

asyncio.run(main())
```

Antes do 3.5, era necessário usar decoradores como `@asyncio.coroutine` e `yield from`. A nova sintaxe é muito mais legível e intuitiva.

Programação assíncrona é essencial para aplicações que fazem muitas operações de I/O (requests HTTP, consultas a banco de dados, leitura de arquivos). Em vez de bloquear a thread esperando a resposta, o `await` libera o controle para outras tarefas.

## Type Hints

O Python 3.5 introduziu o módulo `typing` que permite anotar tipos nas suas funções:

```python
def saudacao(nome: str) -> str:
    return f"Olá, {nome}!"

def somar(a: int, b: int) -> int:
    return a + b

# Listas e dicionários com tipos
from typing import List, Dict

def processar_nomes(nomes: List[str]) -> Dict[str, int]:
    return {nome: len(nome) for nome in nomes}
```

É importante notar que **type hints são opcionais**. O Python continua sendo dinamicamente tipado. As anotações servem para:

- **Documentação**: deixa claro o que a função espera e retorna
- **Ferramentas**: IDEs e linters podem detectar erros antes da execução
- **mypy**: ferramenta que verifica os tipos estaticamente

```bash
pip install mypy
mypy meu_script.py
```

## Matrix Multiplication Operator (@)

O operador `@` foi introduzido para multiplicação de matrizes, algo muito usado em ciência de dados e machine learning:

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Antes do Python 3.5
C = A.dot(B)

# Com o operador @
C = A @ B
```

Isso torna expressões matemáticas muito mais legíveis, especialmente quando há múltiplas multiplicações encadeadas.

## Outras novidades

**`os.scandir()`** — uma alternativa mais rápida ao `os.listdir()`:

```python
import os

# Mais eficiente que os.listdir()
for entrada in os.scandir("/meu/diretorio"):
    if entrada.is_file():
        print(entrada.name)
```

**`zipapp`** — permite criar executáveis Python a partir de scripts:

```bash
python -m zipapp meu_app -o meu_app.pyz
```

**Format strings (f-strings)** chegaram no Python 3.6, mas as mudanças no 3.5 foram o alicerce para muitas das melhorias seguintes.

## Dicas para migração

1. Use a ferramenta `2to3` que vem com o Python para analisar seu código:

```bash
2to3 -w meu_script.py
```

2. Teste tudo. O `pytest` funciona muito bem para validar se a migração não quebrou nada:

```bash
pip install pytest
pytest
```

3. Se precisar manter compatibilidade com Python 2 por algum tempo, use `six` ou `future` como ponte.

## Conclusão

Python 3.5 trouxe recursos que justificam plenamente a migração. O async/await, type hints e o operador `@` são features que melhoram muito a qualidade e legibilidade do código. Se ainda não migrou, comece agora — seu código futuro agradeça.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
