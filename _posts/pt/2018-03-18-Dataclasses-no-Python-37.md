---
layout: post
author: Wellington Pardim
title:  "Dataclasses no Python 3.7: código mais limpo e Pythonico"
categories: "python"
position: 12
---

O Python 3.7 trouxe os **dataclasses**, uma forma mais simples e elegante de criar classes que são basicamente containers de dados. Se você já escreveu dezenas de `__init__`, `__repr__` e `__eq__` manualmente, essa feature vai mudar a sua vida.

## O problema: boilerplate em classes

Considere uma classe simples para representar um usuário:

```python
class Usuario:
    def __init__(self, nome: str, email: str, idade: int, ativo: bool = True):
        self.nome = nome
        self.email = email
        self.idade = idade
        self.ativo = ativo

    def __repr__(self):
        return f"Usuario(nome={self.nome!r}, email={self.email!r}, idade={self.idade}, ativo={self.ativo})"

    def __eq__(self, other):
        if not isinstance(other, Usuario):
            return NotImplemented
        return (self.nome == other.nome and
                self.email == other.email and
                self.idade == other.idade and
                self.ativo == other.ativo)
```

30 linhas para uma classe simples. Com dataclasses:

```python
from dataclasses import dataclass

@dataclass
class Usuario:
    nome: str
    email: str
    idade: int
    ativo: bool = True
```

**6 linhas.** O Python gera automaticamente `__init__`, `__repr__`, `__eq__` e outros métodos.

## Como funciona

O decorador `@dataclass` analisa as anotações de tipo da classe e gera métodos especiais. Ao criar uma instância:

```python
user = Usuario(nome="Wellington", email="well@email.com", idade=30)
print(user)  # Usuario(nome='Wellington', email='well@email.com', idade=30, ativo=True)

# Igualdade funciona automaticamente
user2 = Usuario(nome="Wellington", email="well@email.com", idade=30)
print(user == user2)  # True
```

## Valores padrão

Campos com valores padrão devem vir depois dos campos sem padrão:

```python
@dataclass
class Config:
    host: str
    port: int
    debug: bool = False
    max_connections: int = 100
```

Para valores padrão mutáveis (listas, dicts), use `field(default_factory=...)`:

```python
from dataclasses import dataclass, field

@dataclass
class Time:
    nome: str
    jogadores: list = field(default_factory=list)
    titulos: dict = field(default_factory=dict)

# Cada instância terá sua própria lista e dict
time1 = Time("Flamengo")
time2 = Time("Vasco")
time1.jogadores.append("Jogador 1")
print(time2.jogadores)  # [] — não afetado
```

## `field()` para controle fino

O `field()` oferece opções avançadas:

```python
from dataclasses import dataclass, field

@dataclass
class Produto:
    nome: str
    preco: float
    # Não aparece no __repr__
    _estoque: int = field(repr=False, default=0)
    # Não participa da comparação (__eq__)
    _cache: dict = field(compare=False, default_factory=dict)
    # Valor calculado no __post_init__
    preco_com_imposto: float = field(init=False, default=0.0)

    def __post_init__(self):
        self.preco_com_imposto = self.preco * 1.1
```

## `__post_init__`

O método `__post_init__` é chamado após o `__init__` gerado. Útil para validações ou cálculos:

```python
@dataclass
class Retangulo:
    largura: float
    altura: float
    area: float = field(init=False)

    def __post_init__(self):
        if self.largura <= 0 or self.altura <= 0:
            ValueError("Dimensões devem ser positivas")
        self.area = self.largura * self.altura
```

## Dataclasses imutáveis

Use `frozen=True` para criar objetos que não podem ser modificados após a criação:

```python
@dataclass(frozen=True)
class Ponto:
    x: float
    y: float

p = Ponto(1.0, 2.0)
p.x = 5.0  # Erro! FrozenInstanceError
```

Dataclasses frozen também são **hashable**, o que permite usá-las como chaves de dicionário ou em sets:

```python
pontos = {Ponto(1, 2), Ponto(3, 4), Ponto(1, 2)}
print(len(pontos))  # 2 — duplicatas removidas
```

## Ordenação

Use `order=True` para gerar métodos de comparação (`__lt__`, `__le__`, `__gt__`, `__ge__`):

```python
@dataclass(order=True)
class Tarefa:
    prioridade: int
    nome: str

tarefas = [
    Tarefa(3, "Baixa prioridade"),
    Tarefa(1, "Alta prioridade"),
    Tarefa(2, "Média prioridade"),
]

for t in sorted(tarefas):
    print(f"[{t.prioridade}] {t.nome}")
# [1] Alta prioridade
# [2] Média prioridade
# [3] Baixa prioridade
```

## Comparação: dataclass vs namedtuple vs attrs

```python
# namedtuple — imutável, sem métodos customizados
from collections import namedtuple
PontoNT = namedtuple("PontoNT", ["x", "y"])

# dataclass — flexível, mutável por padrão, type hints nativos
@dataclass
class PontoDC:
    x: float
    y: float

# attrs — biblioteca externa, mais opções (validação, conversão)
# import attr
# @attr.s
# class PontoAttr:
#     x = attr.ib()
#     y = attr.ib()
```

| Feature | namedtuple | dataclass | attrs |
|---------|-----------|-----------|-------|
| Imutável por padrão | Sim | Não | Não |
| Type hints nativos | Não | Sim | Opcional |
| Customização fácil | Limitada | Boa | Excelente |
| Dependência externa | Não | Não | Sim |

## Exemplo real: modelando responses de API

```python
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class Endereco:
    rua: str
    cidade: str
    estado: str
    cep: str

@dataclass
class UsuarioAPI:
    id: int
    nome: str
    email: str
    enderecos: List[Endereco] = field(default_factory=list)
    telefone: Optional[str] = None

    @property
    def endereco_principal(self) -> Optional[Endereco]:
        return self.enderecos[0] if self.enderecos else None

# Simulando uma response de API
dados = {
    "id": 1,
    "nome": "Wellington",
    "email": "well@email.com",
    "enderecos": [
        {"rua": "Rua A", "cidade": "SP", "estado": "SP", "cep": "01000-000"}
    ]
}

# Convertendo de dict
user = UsuarioAPI(
    id=dados["id"],
    nome=dados["nome"],
    email=dados["email"],
    enderecos=[Endereco(**e) for e in dados["enderecos"]]
)
print(user)
```

## Outras novidades do Python 3.7

Além dos dataclasses, o 3.7 trouxe:

- **`breakpoint()`** — função nativa para debug, substituindo `import pdb; pdb.set_trace()`
- **Dicionários ordenados** — `dict` agora mantém ordem de inserção (comportamento garantido)
- **`importlib.resources`** — acesso a recursos dentro de pacotes
- **Pedidos de melhoria (PEP)** — vários PEPs que melhoraram o typing

## Conclusão

Dataclasses eliminam o boilerplate de classes de dados no Python. Para a maioria dos casos onde você criaria uma classe com `__init__` e `__repr__`, um dataclass é a escolha certa. Comece a usá-los nos seus projetos — você vai escrever menos código e ganhar mais legibilidade.

Combine com `frozen=True` para objetos imutáveis e `order=True` para ordenação, e terá classes poderosas com mínimo esforço.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
