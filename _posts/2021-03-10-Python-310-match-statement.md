---
layout: post
author: Wellington Pardim
title:  "Python 3.10: match statement e type hints avançados"
categories: "python"
position: 24
---

O Python 3.10 é uma das releases mais significativas dos últimos anos. O **match statement** (structural pattern matching) é uma feature que muda a forma como escrevemos código condicional. Além disso, houve melhorias importantes em type hints e mensagens de erro.

## Match Statement

O `match` é equivalente ao `switch` de outras linguagens, mas muito mais poderoso. Ele faz **pattern matching estrutural** — não apenas compara valores, mas desestrutura dados.

### Básico

```python
def status_http(codigo):
    match codigo:
        case 200:
            return "OK"
        case 301:
            return "Moved Permanently"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return f"Código desconhecido: {codigo}"

print(status_http(200))  # OK
print(status_http(404))  # Not Found
print(status_http(999))  # Código desconhecido: 999
```

O `_` é o wildcard — captura qualquer valor que não casou com os outros patterns.

### Múltiplos valores

```python
def eh_sucesso(codigo):
    match codigo:
        case 200 | 201 | 204:
            return True
        case _:
            return False
```

### Pattern matching com tipos

```python
def processar(evento):
    match evento:
        case {"tipo": "clique", "x": x, "y": y}:
            print(f"Clique em ({x}, {y})")
        case {"tipo": "tecla", "tecla": tecla}:
            print(f"Tecla pressionada: {tecla}")
        case {"tipo": "scroll", "delta": delta}:
            print(f"Scroll: {delta}")
        case _:
            print("Evento desconhecido")

processar({"tipo": "clique", "x": 100, "y": 200})
# Clique em (100, 200)

processar({"tipo": "tecla", "tecla": "Enter"})
# Tecla pressionada: Enter
```

### Desestruturação de listas

```python
def processar_ponto(ponto):
    match ponto:
        case (0, 0):
            print("Origem")
        case (x, 0):
            print(f"No eixo X: {x}")
        case (0, y):
            print(f"No eixo Y: {y}")
        case (x, y):
            print(f"Ponto ({x}, {y})")

processar_ponto((0, 0))    # Origem
processar_ponto((5, 0))    # No eixo X: 5
processar_ponto((3, 4))    # Ponto (3, 4)
```

### Guards (condições extras)

```python
def classificar_nota(nota):
    match nota:
        case n if n >= 9:
            return "Excelente"
        case n if n >= 7:
            return "Bom"
        case n if n >= 5:
            return "Regular"
        case _:
            return "Insuficiente"
```

### Exemplo real: processando comandos

```python
def executar_comando(comando):
    match comando.split():
        case ["sair"]:
            print("Saindo...")
            return False
        case ["ler", arquivo]:
            print(f"Lendo {arquivo}")
        case ["escrever", arquivo, *linhas]:
            conteudo = " ".join(linhas)
            print(f"Escrevendo em {arquivo}: {conteudo}")
        case ["ajuda"]:
            print("Comandos: ler, escrever, sair, ajuda")
        case _:
            print("Comando não reconhecido")
    return True

executar_comando("ler dados.txt")        # Lendo dados.txt
executar_comando("escrever out.txt Olá") # Escrevendo em out.txt: Olá
```

## Type hints: Union com `|`

Antes do 3.10:

```python
from typing import Union

def processar(valor: Union[str, int]) -> Union[str, int]:
    ...
```

Agora:

```python
def processar(valor: str | int) -> str | int:
    ...
```

Funciona em qualquer contexto:

```python
def buscar(id: int | str) -> dict | None:
    if isinstance(id, int):
        return db.get_by_id(id)
    return db.get_by_name(id)
```

## `TypeAlias`

Para definir aliases de tipo de forma explícita:

```python
from typing import TypeAlias

Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[Vector]

def multiplicar(matriz: Matrix, vetor: Vector) -> Vector:
    return [sum(a * b for a, b in zip(linha, vetor)) for linha in matriz]
```

## Melhorias nas mensagens de erro

O Python 3.10 mostra exatamente onde o erro ocorre:

```python
# Antes do 3.0: mensagem vaga
dicionario = {"a": 1, "b": 2}
print(dicionario["c"])

# Python 3.10: mostra a expressão exata
# Traceback (most recent call last):
#   File "teste.py", line 2, in <module>
#     print(dicionario["c"])
#           ~~~~~~~~~~^^^^^
# KeyError: 'c'
```

O Python agora sublinha a parte exata da expressão que falhou.

## `ParamSpec` para decorators

Novo tipo para anotar decorators que preservam a assinatura da função:

```python
from typing import ParamSpec, TypeVar, Callable

P = ParamSpec("P")
R = TypeVar("R")

def log(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Chamando {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def somar(a: int, b: int) -> int:
    return a + b

# O type checker sabe que somar aceita (int, int) e retorna int
```

## Outras melhorias

- **`zip()` com `strict=True`**: erro se as listas tiverem tamanhos diferentes
- **`int.bit_count()`**: conta bits 1 em um inteiro
- **`os.get_terminal_size()`** melhorado

```python
# zip strict
nomes = ["Ana", "Bob"]
idades = [25, 30, 35]

# Sem strict: silenciosamente ignora o 35
list(zip(nomes, idades))  # [("Ana", 25), ("Bob", 30)]

# Com strict: lança ValueError
list(zip(nomes, idades, strict=True))  # ValueError
```

## Conclusão

O Python 3.10 é um marco na evolução da linguagem. O `match` statement traz pattern matching — algo antes restrito a linguagens funcionais como Haskell e Elixir — para o Python de forma acessível. A sintaxe `|` para Union types torna as anotações mais legíveis.

Se você ainda não experimentou o match statement, comece com exemplos simples (como o status HTTP) e evolua para patterns mais complexos. É uma ferramenta que muda a forma como você pensa sobre condicionais.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
