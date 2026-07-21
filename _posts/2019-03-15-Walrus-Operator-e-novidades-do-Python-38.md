---
layout: post
author: Wellington Pardim
title:  "Walrus Operator e novidades do Python 3.8"
categories: "python"
position: 16
---

O Python 3.8 trouxe uma das features mais controversas dos últimos anos: o **walrus operator** (`:=`). Além dele, houve melhorias significativas em funções, f-strings e performance. Vamos conhecer tudo.

## Walrus Operator (:=)

O operador `:=` (apelidado de "walrus" porque parece uma morsa de lado) permite **atribuir e retornar um valor em uma mesma expressão**. Ele foi definido na PEP 572.

Onde antes você escrevia:

```python
linha = input("> ")
while linha != "sair":
    print(f"Você digitou: {linha}")
    linha = input("> ")
```

Agora pode escrever:

```python
while (linha := input("> ")) != "sair":
    print(f"Você digitou: {linha}")
```

A variável `linha` é atribuída **e** comparada na mesma linha.

### Usos práticos

**Filtrando e processando em list comprehensions:**

```python
# Sem walrus: calcula len() duas vezes
nomes = [nome for nome in lista if len(nome) > 5]
print([nome.upper() for nome in nomes])

# Com walrus: calcula len() uma vez
nomes_grandes = [
    nome.upper()
    for nome in lista
    if (tamanho := len(nome)) > 5
]
```

**Evitando cálculo duplicado:**

```python
# Sem walrus
resultado = funcao_cara(dados)
if resultado > limite:
    print(f"Resultado: {resultado}")

# Com walrus
if (resultado := funcao_cara(dados)) > limite:
    print(f"Resultado: {resultado}")
```

**Em loops com condição:**

```python
# Ler dados até encontrar vazio
while (dado := ler_proximo()) is not None:
    processar(dado)
```

**Em filtros com logging:**

```python
erros = [
    linha
    for linha in log
    if (match := regex_erro.search(linha))
]
```

### Quando NÃO usar

Evite o walrus operator quando:
- A expressão é simples demais (`x := 5` é desnecessário)
- Reduz a legibilidade em vez de aumentar
- O colega que vai manter seu código não vai entender

## positional-only parameters

O Python 3.8 permite definir parâmetros que **só podem ser passados por posição**, usando `/`:

```python
def calcular(x, y, /, operacao="soma"):
    if operacao == "soma":
        return x + y
    return x * y

# Correto
calcular(3, 4)
calcular(3, 4, operacao="multiplica")

# Erro! Não pode passar por nome
calcular(x=3, y=4)  # TypeError
```

Isso é útil para bibliotecas que querem liberdade para renomear parâmetros sem quebrar código existente.

## `f` em f-strings para debug

Uma pequena melhoria que salva muito tempo:

```python
nome = "Wellington"
idade = 30

# Antes do 3.8
print(f"nome={nome!r}, idade={idade!r}")

# Python 3.8: o = faz o debug automaticamente
print(f"{nome=}, {idade=}")
# Saída: nome='Wellington', idade=30

# Com expressões
lista = [1, 2, 3]
print(f"{len(lista)=}")
# Saída: len(lista)=3
```

## `math.prod`

Nova função para multiplicar todos os elementos de uma sequência:

```python
import math

numeros = [2, 3, 4, 5]
print(math.prod(numeros))  # 120

# Equivalente a
from functools import reduce
import operator
reduce(operator.mul, numeros)
```

## `typing.TypedDict`

Para dicionários com estrutura fixa:

```python
from typing import TypedDict

class UsuarioDict(TypedDict):
    nome: str
    email: str
    idade: int

# Type checker verifica os tipos
user: UsuarioDict = {
    "nome": "Wellington",
    "email": "well@email.com",
    "idade": 30,
}
```

## Outras melhorias

- **`statistics.fmean()`**: média de ponto flutuante, mais rápida que `mean()`
- **`shlex.join()`**: junta lista de strings em comando de shell seguro
- **`importlib.metadata`**: acesso a metadados de pacotes instalados
- **Melhorias no `asyncio`**: `asyncio.run()` estável, `asyncio.Task.cancel()` melhorado

## Conclusão

O Python 3.8 é uma release que melhora a ergonomia da linguagem. O walrus operator é controverso, mas quando usado com moderação, elimina código duplicado e torna loops mais limpos. As positional-only parameters e o `f"{var=}"` são melhorias que todo mundo usa naturalmente.

Se você ainda não atualizou, vale a pena — especialmente pelo `f"{var=}"` que economiza muito tempo em debug.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
