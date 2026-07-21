---
layout: post
author: Wellington Pardim
title:  "f-strings and other Python 3.6 features"
categories: "python"
position: 8
lang: en
translation: /2017/02/10/f-strings-e-novidades-do-Python-36.html
---

Python 3.6 brought several improvements that made code more readable and Pythonic. The most popular is **f-strings**, but there are other features worth knowing.

## f-strings: finally readable formatting

Before Python 3.6, string formatting was verbose:

```python
name = "Wellington"
age = 30

# Old % method
print("My name is %s and I'm %d years old" % (name, age))

# .format()
print("My name is {} and I'm {} years old".format(name, age))

# .format() with indexes
print("My name is {0} and I'm {1} years old".format(name, age))
```

With f-strings, just prefix the string with `f` and use curly braces:

```python
name = "Wellington"
age = 30

print(f"My name is {name} and I'm {age} years old")
```

Simple, direct, and readable.

## Expressions inside f-strings

The real power of f-strings is that you can put **any Python expression** inside the curly braces:

```python
# Method calls
name = "wellington"
print(f"Name: {name.upper()}")  # WELLINGTON

# Math operations
price = 49.90
quantity = 3
print(f"Total: ${price * quantity:.2f}")  # $149.70

# Conditionals
age = 17
print(f"Adult: {'Yes' if age >= 18 else 'No'}")  # No

# Accessing attributes
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

user = User("Wellington", "well@email.com")
print(f"Contact: {user.name} <{user.email}>")
```

## Format specifiers

You can control formatting with specifiers after `:`:

```python
# Decimal places
pi = 3.14159265
print(f"Pi: {pi:.2f}")  # 3.14

# Zero padding
code = 42
print(f"Code: {code:05d}")  # 00042

# Thousands separator
population = 211_000_000
print(f"Population: {population:,}")  # 211,000,000

# Percentage
rate = 0.857
print(f"Rate: {rate:.1%}")  # 85.7%

# Alignment
for name, grade in [("Ana", 9.5), ("Bob", 7.2), ("Carlos", 8.8)]:
    print(f"{name:<10} {grade:>5.1f}")
# Ana           9.5
# Bob           7.2
# Carlos        8.8
```

## The secrets module

Python 3.6 introduced the `secrets` module for cryptographically secure random data. Unlike `random` (which is predictable), `secrets` is suitable for passwords, tokens, and keys:

```python
import secrets

# Hex token of 32 bytes
token = secrets.token_hex(32)
print(token)  # a3f8b2c1d4e5...

# URL-safe token
url_token = secrets.token_urlsafe(32)
print(url_token)

# Secure random choice
pwd_chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$"
password = "".join(secrets.choice(pwd_chars) for _ in range(16))
print(password)
```

This module is essential for any application dealing with security. In the [hashing article](/2016/09/10/Ferramenta-CLI-em-Python-para-Hashing-de-Senhas.html) I use `secrets` to generate salts.

## Variable annotations

Python 3.6 introduced PEP 526, allowing type annotations on variables:

```python
# Simple annotations
name: str = "Wellington"
age: int = 30
active: bool = True

# Complex types
from typing import List, Dict

users: List[str] = ["Ana", "Bob"]
config: Dict[str, int] = {"timeout": 30, "retries": 3}

# Annotation without initial value
balance: float
balance = 100.50
```

Like function type hints (introduced in [Python 3.5](/2016/06/20/Python-35-o-que-mudou-e-por-que-migrar.html)), variable annotations are optional and serve for documentation and static verification with `mypy`.

## Async generators

Python 3.6 allowed `await` inside generators, creating async generators:

```python
import asyncio

async def counter(limit):
    i = 0
    while i < limit:
        yield i
        await asyncio.sleep(0.5)
        i += 1

async def main():
    async for number in counter(5):
        print(number)

asyncio.run(main())
```

Useful for consuming data asynchronously from streams, websockets, or paginated APIs.

## Other minor improvements

- **`__init_subclass__`**: hook called when a subclass is created, useful for frameworks
- **Underscores in numeric literals**: `1_000_000` instead of `1000000` for readability
- **`asyncio` and `await` at module level**: no longer need to manually create an event loop

## Conclusion

Python 3.6 was a release that significantly improved the coding experience. f-strings alone justify the migration — the readability they bring is unmatched. Combine that with the `secrets` module and variable annotations, and we have a very solid release.

If you're still on Python 3.5 or earlier, the upgrade is well worth it.

If I said anything wrong, corrections and suggestions are welcome.
