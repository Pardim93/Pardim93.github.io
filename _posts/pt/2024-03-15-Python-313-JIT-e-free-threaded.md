---
layout: post
author: Wellington Pardim
title:  "Python 3.13: JIT experimental e free-threaded mode"
categories: "python"
position: 36
---

O Python 3.13 é historicamente significativo: é o primeiro release com um **JIT compiler experimental** e suporte a **free-threaded mode** (sem GIL). Essas são as mudanças mais ambiciosas na história do CPython.

## Free-threaded mode (no GIL)

O **GIL** (Global Interpreter Lock) é o que impedia o Python de usar múltiplos cores para threads. O Python 3.13 introduz um modo experimental onde o GIL pode ser desabilitado (PEP 703).

### O que é o GIL?

O GIL é um mutex que permite que apenas uma thread execute bytecode Python por vez:

```python
import threading
import time

contador = 0

def incrementar():
    global contador
    for _ in range(1_000_000):
        contador += 1

# Com GIL: threads rodam uma por vez
# O resultado é correto, mas não há paralelismo real
threads = [threading.Thread(target=incrementar) for _ in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

### Free-threaded Python

```bash
# Instale o Python 3.13 free-threaded
# No build, habilite:
./configure --disable-gil
make
```

```python
# Com free-threaded mode:
# - Threads rodam em paralelo de verdade
# - Cuidado com race conditions!
# - Use locks quando necessário

import threading

contador = 0
lock = threading.Lock()

def incrementar():
    global contador
    for _ in range(1_000_000):
        with lock:
            contador += 1
```

### Status atual

- **Experimental**: não use em produção ainda
- **ABI diferente**: binários precisam ser compilados especificamente para free-threaded
- **Performance**: pode ser mais lento para código single-thread devido ao overhead de locks granulares
- **Bibliotecas**: muitas ainda não testadas com free-threaded mode

## JIT Compiler (copy-and-patch)

O Python 3.13 inclui um **JIT compiler experimental** baseado na técnica "copy-and-patch" (PEP 744).

### Como funciona

O JIT compila bytecode Python em código nativo usando templates pré-compilados:

```
Python bytecode → Copy-and-patch JIT → Machine code
```

O "copy-and-patch" copia templates de código nativo e os "patcha" com valores específicos. É mais rápido que um interpretador puro, mas menos otimizado que um JIT agressivo como o do Java.

### Status

- **Experimental**: precisa ser habilitado na compilação
- **Ganhos modestos**: ~5-10% em benchmarks iniciais
- **Roadmap**: melhorias incrementais nos próximos releases (3.14, 3.15...)

## Melhorias no REPL

O REPL do Python 3.13 recebeu melhorias significativas:

```python
# Colors no REPL (habilitado por padrão)
>>> def hello():
...     print("Hello!")
... 
>>> hello()
Hello!

# Multi-line editing melhorado
>>> class Foo:
...     def bar(self):
...         return 42
... 

# History compartilhado entre sessões
```

## `warnings.deprecated`

Novo decorator para marcar APIs como deprecated:

```python
from warnings import deprecated

@deprecated("Use nova_funcao() em vez disso")
def funcao_antiga():
    pass

# Usar empresta um warning
funcao_antiga()
# DeprecationWarning: Use nova_funcao() em vez disso
```

## `dbm.sqlite3` como padrão

O módulo `dbm` agora usa SQLite como backend padrão, substituindo o antigo Berkeley DB:

```python
import dbm

# Agora usa SQLite internamente
with dbm.open("cache", "c") as db:
    db["chave"] = "valor"
    print(db["chave"])  # b"valor"
```

## `os` e `pathlib` melhorias

```python
from pathlib import Path

# Path.walk() confirmado (desde 3.12)
for dirpath, dirnames, filenames in Path("/projeto").walk():
    print(f"{dirpath}: {len(filenames)} arquivos")

# os.listdrives() no Windows
import os
drives = os.listdrives()
print(drives)  # ['C:\\', 'D:\\']
```

## `asyncio` improvements

```python
import asyncio

# asyncio.Barrier para sincronização
async def worker(barrier, nome):
    print(f"{nome} trabalhando...")
    await asyncio.sleep(1)
    await barrier.wait()
    print(f"{nome} passou da barreira!")

async def main():
    barrier = asyncio.Barrier(3)
    await asyncio.gather(
        worker(barrier, "A"),
        worker(barrier, "B"),
        worker(barrier, "C"),
    )
```

## `typing` improvements

```python
from typing import TypeIs

def eh_string(valor: object) -> TypeIs[str]:
    return isinstance(valor, str)

def processar(valor: object):
    if eh_string(valor):
        # Type checker sabe que valor é str aqui
        print(valor.upper())
```

## Conclusão

O Python 3.13 é o início de uma nova era. O free-threaded mode e o JIT compiler são features que vão amadurecer nos próximos 3-5 releases, eventualmente transformando o Python em uma linguagem verdadeiramente paralela e mais rápida.

Para hoje, use o 3.13 para experimentar. Para produção, o 3.12 continua sendo a escolha estável. Mas fique atento — o futuro do Python está sendo construído agora.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
