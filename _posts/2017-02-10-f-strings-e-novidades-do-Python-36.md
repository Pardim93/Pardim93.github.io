---
layout: post
author: Wellington Pardim
title:  "f-strings e outras novidades do Python 3.6"
categories: "python"
position: 8
---

O Python 3.6 trouxe várias melhorias que tornaram o código mais legível e Pythonico. A mais popular delas são as **f-strings**, mas há outros recursos que valem a pena conhecer.

## f-strings: formatação finalmente legível

Antes do Python 3.6, a formatação de strings era verbosa:

```python
nome = "Wellington"
idade = 30

# Método antigo com %
print("Meu nome é %s e tenho %d anos" % (nome, idade))

# .format()
print("Meu nome é {} e tenho {} anos".format(nome, idade))

# .format() com índices
print("Meu nome é {0} e tenho {1} anos".format(nome, idade))
```

Com f-strings, basta prefixar a string com `f` e usar chaves:

```python
nome = "Wellington"
idade = 30

print(f"Meu nome é {nome} e tenho {idade} anos")
```

Simples, direto e legível.

## Expressões dentro de f-strings

O grande poder das f-strings é que você pode colocar **qualquer expressão Python** dentro das chaves:

```python
# Chamadas de método
nome = "wellington"
print(f"Nome: {nome.upper()}")  # WELLINGTON

# Operações matemáticas
preco = 49.90
quantidade = 3
print(f"Total: R$ {preco * quantidade:.2f}")  # R$ 149.70

# Condicionais
idade = 17
print(f"Maior de idade: {'Sim' if idade >= 18 else 'Não'}")  # Não

# Acessando atributos
class Usuario:
    def __init__(self, nome, email):
        self.nome = nome
        self.email = email

user = Usuario("Wellington", "well@email.com")
print(f"Contato: {user.nome} <{user.email}>")
```

## Format specifiers

Você pode controlar a formatação com especificadores após o `:`:

```python
# Casas decimais
pi = 3.14159265
print(f"Pi: {pi:.2f}")  # 3.14

# Preenchimento com zeros
codigo = 42
print(f"Código: {codigo:05d}")  # 00042

# Separador de milhares
populacao = 211_000_000
print(f"População: {populacao:,}")  # 211,000,000

# Porcentagem
acertos = 0.857
print(f"Aproveitamento: {acertos:.1%}")  # 85.7%

# Alinhamento
for nome, nota in [("Ana", 9.5), ("Bob", 7.2), ("Carlos", 8.8)]:
    print(f"{nome:<10} {nota:>5.1f}")
# Ana           9.5
# Bob           7.2
# Carlos        8.8
```

## O módulo secrets

O Python 3.6 introduziu o módulo `secrets` para geração de dados criptograficamente seguros. Diferente do `random` (que é previsível), o `secrets` é adequado para senhas, tokens e chaves:

```python
import secrets

# Token hexadecimal de 32 bytes
token = secrets.token_hex(32)
print(token)  # a3f8b2c1d4e5...

# Token URL-safe
url_token = secrets.token_urlsafe(32)
print(url_token)

# Escolha aleatória segura
senha_chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$"
senha = "".join(secrets.choice(senha_chars) for _ in range(16))
print(senha)
```

Esse módulo é essencial para qualquer aplicação que lide com segurança. No [artigo sobre hashing](/2016/09/10/Ferramenta-CLI-em-Python-para-Hashing-de-Senhas.html) eu uso `secrets` para gerar salts.

## Anotações de variáveis

O Python 3.6 introduziu PEP 526 que permite anotar tipos em variáveis:

```python
# Anotações simples
nome: str = "Wellington"
idade: int = 30
ativo: bool = True

# Tipos complexos
from typing import List, Dict

usuarios: List[str] = ["Ana", "Bob"]
config: Dict[str, int] = {"timeout": 30, "retries": 3}

# Anotação sem valor inicial
saldo: float
saldo = 100.50
```

Assim como as type hints de funções (introduzidas no [Python 3.5](/2016/06/20/Python-35-o-que-mudou-e-por-que-migrar.html)), as anotações de variáveis são opcionais e servem para documentação e verificação estática com `mypy`.

## Async generators

O Python 3.6 permitiu `await` dentro de geradores, criando os async generators:

```python
import asyncio

async def contador(limite):
    i = 0
    while i < limite:
        yield i
        await asyncio.sleep(0.5)
        i += 1

async def main():
    async for numero in contador(5):
        print(numero)

asyncio.run(main())
```

Útil para consumir dados de forma assíncrona em streams, websockets ou APIs paginadas.

## Outras melhorias menores

- **`__init_subclass__`**: hook chamado quando uma subclasse é criada, útil para frameworks
- **Underscores em literais numéricos**: `1_000_000` em vez de `1000000` para legibilidade
- **`asyncio` e `await` em nível de módulo**: não é mais necessário criar um loop de eventos manualmente

## Conclusão

O Python 3.6 foi uma release que melhorou significativamente a experiência de escrever código. As f-strings sozinhas já justificam a migração — a legibilidade que elas trazem é incomparável. Combine isso com o módulo `secrets` e as anotações de variáveis, e temos um release muito sólido.

Se você ainda está no Python 3.5 ou anterior, vale muito a pena atualizar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
