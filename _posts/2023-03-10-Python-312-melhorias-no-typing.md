---
layout: post
author: Wellington Pardim
title:  "Python 3.12: melhorias no typing e perf counters"
categories: "python"
position: 32
---

O Python 3.12 continua a tradição de melhorias incrementais focadas em performance e developer experience. As principais novidades são type parameter syntax, better f-strings, e novas ferramentas de profiling.

## Type Parameter Syntax

O Python 3.12 introduz uma sintaxe mais limpa para definir generics (PEP 695):

```python
# Antes do 3.12
from typing import TypeVar, Generic

T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: list[T] = []
    def push(self, item: T) -> None:
        self.items.append(item)
    def pop(self) -> T:
        return self.items.pop()

# Python 3.12: sintaxe nativa
class Stack[T]:
    def __init__(self) -> None:
        self.items: list[T] = []
    def push(self, item: T) -> None:
        self.items.append(item)
    def pop(self) -> T:
        return self.items.pop()
```

Para funções:

```python
# Antes
from typing import TypeVar

T = TypeVar("T")

def first[T](lst: list[T]) -> T | None:
    return lst[0] if lst else None

# Python 3.12
def first[T](lst: list[T]) -> T | None:
    return lst[0] if lst else None
```

Com constraints:

```python
# Type com bound
def max_value[T: Comparable](a: T, b: T) -> T:
    return a if a > b else b

# Type com constraints explícitos
type Number = int | float | complex
```

## `type` statement

Novo statement para definir type aliases:

```python
# Antes
from typing import TypeAlias
Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[Vector]

# Python 3.12
type Vector = list[float]
type Matrix = list[Vector]
```

Funciona com generics:

```python
type Result[T] = T | Exception
type Pair[T, U] = tuple[T, U]
```

## Better f-strings

F-strings agora suportam expressões mais complexas:

```python
# Multiline f-strings
mensagem = f"""
    Usuário: {user.nome}
    E-mail: {user.email}
    Status: {"Ativo" if user.ativo else "Inativo"}
"""

# Comentários dentro de f-strings
valor = 42
print(f"O valor é {
    valor  # Isto é um comentário
}")

# Backslashes (não era possível antes)
print(f"Nova linha: {
    'linha1\nlinha2'
}")
```

## Perf counters

Novas funções de alta resolução para profiling:

```python
import time

# time.perf_counter() já existia
# time.perf_counter_ns() retorna em nanossegundos
inicio = time.perf_counter_ns()
# ... código ...
fim = time.perf_counter_ns()
print(f"Tempo: {(fim - inicio) / 1_000_000:.2f}ms")

# time.monotonic_ns()
# Garantido monotônico (não volta atrás com ajustes de relógio)
```

## `itertools.batched`

Nova função para processar dados em lotes:

```python
from itertools import batched

dados = range(10)

for batch in batched(dados, 3):
    print(batch)
# (0, 1, 2)
# (3, 4, 5)
# (6, 7, 8)
# (9,)

# Útil para processar APIs em lotes
def enviar_em_lote(ids):
    for batch in batched(ids, 100):
        api.enviar_lote(batch)
```

## `pathlib.Path.walk()`

Walking em diretórios de forma mais eficiente:

```python
from pathlib import Path

# Antes: os.walk()
import os
for dirpath, dirnames, filenames in os.walk("/projeto"):
    ...

# Python 3.12: Path.walk()
for dirpath, dirnames, filenames in Path("/projeto").walk():
    for f in filenames:
        print(dirpath / f)
```

## `asyncio.TaskGroup` aprimorado

Melhor tratamento de erros em TaskGroups:

```python
import asyncio

async def tarefa(n):
    await asyncio.sleep(1)
    if n == 3:
        raise ValueError(f"Erro na tarefa {n}")
    return n

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(tarefa(i)) for i in range(5)]
    except* ValueError as eg:
        for exc in eg.exceptions:
            print(f"Capturado: {exc}")

asyncio.run(main())
```

## Subinterpreters

Suporte experimental a subinterpreters (PEP 684), permitindo verdadeiro paralelismo com GIL separado por interpreter:

```python
# Experimental — para extensões C principalmente
import _xxsubinterpreters as interpreters

interp = interpreters.create()
interpreters.run_string(interp, "print('Olá de outro interpreter!')")
interpreters.destroy(interp)
```

## Melhorias no traceback

```python
# Python 3.12 mostra sugestões para typos
class MinhaClasse:
    def metodo_correto(self):
        pass

obj = MinhaClasse()
obj.metodo_coreto()  # AttributeError: 'MinhaClasse' object has no attribute 'metodo_coreto'. Did you mean: 'metodo_correto'?
```

## Outras melhorias

- **`datetime`**: `datetime.fromtimestamp()` com timezone mais precisa
- **`hashlib`**: suporte a SHA-3 via OpenSSL (mais rápido)
- **`sqlite3`**: suporte a `autocommit`
- **`os`**: `os.waitstatus_to_exitcode()` no Windows
- **Buffer protocol**: suporte a `__buffer__` em Python puro

## Conclusão

O Python 3.12 é uma release que poli e refina. A type parameter syntax elimina boilerplate de generics, o `type` statement torna aliases mais claros, e `itertools.batched` é uma daquelas funções que você não sabia que precisava até usar.

Combine com o 3.10 (match) e 3.11 (performance) para ter o melhor Python possível.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
