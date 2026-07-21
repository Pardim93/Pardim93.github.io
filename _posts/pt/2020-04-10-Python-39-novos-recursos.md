---
layout: post
author: Wellington Pardim
title:  "Python 3.9: novos recursos e melhorias no dicionário"
categories: "python"
position: 20
---

O Python 3.9 trouxe operadores novos para dicionários, melhorias no parser, e funções úteis na standard library. É uma release que torna o dia a dia mais produtivo.

## Merge e Update de dicionários com operadores

A feature mais esperada do 3.9 são os operadores `|` e `|=` para dicionários (PEP 584).

Antes:

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}

# Merge (cria novo dict)
merged = {**dict1, **dict2}

# Update (modifica existente)
dict1.update(dict2)
```

Agora:

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}

# Merge com |
merged = dict1 | dict2
print(merged)  # {"a": 1, "b": 3, "c": 4}

# Update com |=
dict1 |= dict2
print(dict1)  # {"a": 1, "b": 3, "c": 4}
```

O `|` cria um novo dicionário (imutável em relação aos originais). O `|=` modifica o dicionário in-place, equivalente ao `.update()`.

Funciona com qualquer mapeável:

```python
from collections import Counter

contagem = Counter({"a": 3, "b": 1})
extra = Counter({"b": 2, "c": 5})

resultado = contagem | extra
print(resultado)  # Counter({"a": 3, "b": 2, "c": 5})
```

## Novo parser (PEG)

O Python 3.9 substituiu o parser LL(1) por um parser baseado em **PEG** (Parsing Expression Grammar). Isso não muda nada para o usuário, mas permite que futuras versões do Python tenham sintaxe mais expressiva.

## Tipos genéricos nas builtins

Agora você pode usar `list`, `dict`, `tuple`, `set` diretamente nas anotações de tipo, sem importar do `typing`:

```python
# Antes do 3.9
from typing import List, Dict, Tuple, Set

def processar(nomes: List[str], config: Dict[str, int]) -> Tuple[str, int]:
    ...

# Python 3.9
def processar(nomes: list[str], config: dict[str, int]) -> tuple[str, int]:
    ...
```

Mesma coisa para `type()`:

```python
# Antes
from typing import List
lista: List[int] = [1, 2, 3]

# Agora
lista: list[int] = [1, 2, 3]
```

## `str.removeprefix()` e `str.removesuffix()`

Finalmente métodos nativos para remover prefixos e sufixos:

```python
arquivo = "relatorio_2020.csv"

# Antes
if arquivo.startswith("relatorio_"):
    nome = arquivo[len("relatorio_"):]
if arquivo.endswith(".csv"):
    nome = arquivo[:-4]

# Agora
nome = arquivo.removeprefix("relatorio_").removesuffix(".csv")
print(nome)  # 2020
```

Útil para processar nomes de arquivos, URLs, e strings com padrões conhecidos:

```python
urls = [
    "https://api.exemplo.com/v1/users",
    "https://api.exemplo.com/v1/posts",
    "https://api.exemplo.com/v2/users",
]

caminhos = [url.removeprefix("https://api.exemplo.com") for url in urls]
print(caminhos)
# ['/v1/users', '/v1/posts', '/v2/users']
```

## `zoneinfo` — fusos horários na stdlib

O Python 3.9 introduziu o módulo `zoneinfo` (PEP 615), eliminando a dependência de `pytz`:

```python
from datetime import datetime
from zoneinfo import ZoneInfo

# Hora atual em São Paulo
sp = ZoneInfo("America/Sao_Paulo")
agora = datetime.now(sp)
print(agora.strftime("%d/%m/%Y %H:%M"))

# Conversão entre fusos
tokyo = ZoneInfo("Asia/Tokyo")
hora_tokyo = agora.astimezone(tokyo)
print(hora_tokyo.strftime("%d/%m/%Y %H:%M"))

# Lista de fusos disponíveis
from zoneinfo import available_timezones
print(len(available_timezones()))  # ~590
```

## `random.Random` com seeds mais seguras

O módulo `random` agora usa o sistema operacional para gerar seeds melhores:

```python
import random

# Gera bytes aleatórios usando entropia do OS
token = random.randbytes(16)
print(token.hex())
```

Para segurança, continue usando `secrets` — mas para simulações e testes, o `random` agora tem seeds melhores.

## Melhorias no `ast`

O módulo `ast` agora suporta `ast.unparse()`, que converte uma árvore AST de volta para código:

```python
import ast

codigo = "x = 1 + 2"
arvore = ast.parse(codigo)

# Modificar a árvore
for node in ast.walk(arvore):
    if isinstance(node, ast.Constant) and node.value == 2:
        node.value = 3

# Converter de volta para código
novo_codigo = ast.unparse(arvore)
print(novo_codigo)  # x = 1 + 3
```

## Outras melhorias

- **`functools.cache`**: simplificação de `lru_cache(maxsize=None)`
- **`math.gcd`** agora aceita múltiplos argumentos: `math.gcd(12, 18, 24)`
- **`isocalendar()`** retorna um `namedtuple` agora
- **`__init_subclass__`** recebe `**kwargs` corretamente

```python
from functools import cache

@cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

## Conclusão

O Python 3.9 é uma release pragmática — resolve dores reais do dia a dia. O operador `|` para dicionários, `removeprefix`/`removesuffix`, e os tipos genéricos nas builtins são features que você vai usar naturalmente, sem precisar pensar.

Se você ainda está no 3.8 ou anterior, a migração é suave e vale muito a pena.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
