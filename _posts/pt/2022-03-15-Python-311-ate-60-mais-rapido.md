---
layout: post
author: Wellington Pardim
title:  "Python 3.11: até 60% mais rápido"
categories: "python"
position: 28
---

O Python 3.11 é a release com o maior ganho de performance da história do CPython — até **60% mais rápido** em benchmarks específicos. Além disso, trouxe melhorias significativas em exceções, typing e stdlib.

## Faster CPython

O projeto **Faster CPython** (liderado por Mark Shannon, com contribuição de Guido van Rossum) está otimizando o CPython incrementalmente. O 3.11 é o primeiro release com as otimizações visíveis.

### O que foi otimizado

1. **Quickening**: bytecode especializado é gerado na primeira execução
2. **Inline caches**: acesso mais rápido a atributos e variáveis locais
3. **Frozen imports**: módulos do stdlib carregam mais rápido
4. **Zero-cost exceptions**: exceções que não são lançadas não custam nada

### Benchmarks

```python
# Benchmark simples: fibonacci recursivo
import time

def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

inicio = time.time()
fib(34)
print(f"Tempo: {time.time() - inicio:.2f}s")

# Python 3.10: ~3.5s
# Python 3.11: ~2.2s (~37% mais rápido)
```

O speedup varia por workload. Código com muitas chamadas de função e acesso a atributos se beneficia mais.

## Exception Groups e `except*`

O Python 3.11 introduziu **Exception Groups** (PEP 654) — a capacidade de lançar e capturar múltiplas exceções simultaneamente. Essencial para programação assíncrona onde múltiplas tasks podem falhar independentemente.

```python
# Exception Group
def processar_lote(itens):
    erros = []
    resultados = []
    for item in itens:
        try:
            resultados.append(processar(item))
        except Exception as e:
            erros.append(e)
    
    if erros:
        raise ExceptionGroup("Erros no lote", erros)
    
    return resultados

# Capturando com except*
try:
    processar_lote(itens)
except* ValueError as eg:
    print(f"Erros de valor: {eg.exceptions}")
except* TypeError as eg:
    print(f"Erros de tipo: {eg.exceptions}")
```

O `except*` é novo — ele captura apenas as exceções do grupo que combinam com o tipo especificado.

## Tracebacks mais informativos

O Python 3.11 mostra **exatamente** onde o erro ocorreu na expressão:

```python
# Python 3.11 mostra a expressão exata
dicionario = {"a": {"b": {"c": 42}}}
print(dicionario["x"]["y"]["z"])

# Traceback (most recent call last):
#   File "teste.py", line 2
#     print(dicionario["x"]["y"]["z"])
#           ~~~~~~~~~~~^^^^^
# KeyError: 'x'
```

O Python sublinha a parte exata da expressão que falhou, em vez de apenas mostrar a linha.

## `tomllib` no stdlib

Leitura de arquivos TOML sem instalar nada:

```python
import tomllib

with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)

print(config["project"]["name"])
```

**Nota**: `tomllib` apenas lê TOML. Para escrever, use a biblioteca `tomli-w`.

## `TaskGroup` para async

```python
import asyncio

async def buscar(url):
    await asyncio.sleep(1)  # simula request
    return f"Dados de {url}"

async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(buscar("https://api1.com"))
        t2 = tg.create_task(buscar("https://api2.com"))
        t3 = tg.create_task(buscar("https://api3.com"))
    
    # Todas completaram (ou uma exceção foi lançada)
    print(t1.result())
    print(t2.result())
    print(t3.result())
```

Se qualquer task falhar, o `TaskGroup` cancela as outras e levanta um `ExceptionGroup`.

## `Self` type

Novo tipo para métodos que retornam a própria instância:

```python
from typing import Self

class Query:
    def where(self, condicao: str) -> Self:
        self._condicoes.append(condicao)
        return self
    
    def limit(self, n: int) -> Self:
        self._limit = n
        return self

# Type checker entende o chaining
resultado = Query().where("idade > 18").limit(10)
```

## `@dataclass_transform`

Para libraries que criam classes similares a dataclasses:

```python
from typing import dataclass_transform

@dataclass_transform()
def my_model(cls):
    # Transformação customizada
    return dataclass(cls)

@my_model
class User:
    name: str
    age: int
```

## Outras melhorias

- **`asyncio.Barrier`**: sincronização entre tasks
- **`math.cbrt()`**: raiz cúbica nativa
- **`random.randint()`** mais rápido
- **`str` e `bytes`** têm `removeprefix`/`removesuffix` confirmados (desde 3.9)
- **Fine-grained error locations**: mensagens de erro mostram coluna exata

## Conclusão

O Python 3.11 é uma release que entrega o que todo mundo queria: **velocidade**. O Faster CPython está apenas começando — o 3.12 e 3.13 continuam as otimizações. Se você ainda está no 3.9 ou 3.10, a migração para o 3.11 é altamente recomendada.

Combine a performance do 3.11 com as features do 3.10 (match statement) e você terá o melhor Python que já existiu.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
