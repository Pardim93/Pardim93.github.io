---
layout: post
author: Wellington Pardim
title:  "Ferramenta CLI em Python para Hashing de Senhas"
categories: "python security"
position: 6
---

Em um [artigo anterior](/2020/02/09/Hashing-e-Encripitacao.html) expliquei a diferença entre hashing e encriptação, e como o salting protege senhas contra ataques de força bruta. Hoje vou colocar esses conceitos em prática construindo uma ferramenta de linha de comando em Python que faz hashing e verificação de senhas.

## Por que construir isso?

Entender como hashing funciona na teoria é importante, mas implementar na prática consolida o conhecimento. Vamos usar o `hashlib` (que vem com o Python) e o `argparse` para criar uma CLI completa.

## O código completo

```python
#!/usr/bin/env python3
"""
hashpass - Ferramenta CLI para hashing e verificação de senhas.
Uso:
    python hashpass.py hash "minhaSenha"
    python hashpass.py verify "minhaSenha" "hash_armazenado"
"""

import argparse
import hashlib
import secrets
import sys


def gerar_salt(tamanho: int = 32) -> str:
    """Gera um salt aleatório criptograficamente seguro."""
    return secrets.token_hex(tamanho)


def hash_senha(senha: str, salt: str = None, algoritmo: str = "sha256") -> dict:
    """
    Gera o hash de uma senha com salt.

    Args:
        senha: a senha em texto plano
        salt: salt opcional (gera um novo se não fornecido)
        algoritmo: algoritmo de hash (padrão: sha256)

    Returns:
        dict com salt, hash e algoritmo usado
    """
    if salt is None:
        salt = gerar_salt()

    senha_com_salt = f"{salt}{senha}".encode("utf-8")
    hash_obj = hashlib.new(algoritmo, senha_com_salt)
    hash_hex = hash_obj.hexdigest()

    return {
        "salt": salt,
        "hash": hash_hex,
        "algoritmo": algoritmo,
    }


def verificar_senha(senha: str, hash_armazenado: str, salt: str,
                     algoritmo: str = "sha256") -> bool:
    """
    Verifica se uma senha corresponde ao hash armazenado.

    Args:
        senha: senha em texto plano para verificar
        hash_armazenado: hash previamente armazenado
        salt: salt usado no hash original
        algoritmo: algoritmo de hash usado

    Returns:
        True se a senha confere, False caso contrário
    """
    resultado = hash_senha(senha, salt, algoritmo)
    return resultado["hash"] == hash_armazenado


def cmd_hash(args):
    """Comando: hashpass hash <senha>"""
    resultado = hash_senha(args.senha, algoritmo=args.algoritmo)
    print(f"Salt:      {resultado['salt']}")
    print(f"Hash:      {resultado['hash']}")
    print(f"Algoritmo: {resultado['algoritmo']}")


def cmd_verify(args):
    """Comando: hashpass verify <senha> <hash> --salt <salt>"""
    if not args.salt:
        print("Erro: o parâmetro --salt é obrigatório para verificação.",
              file=sys.stderr)
        sys.exit(1)

    ok = verificar_senha(args.senha, args.hash, args.salt, args.algoritmo)
    if ok:
        print("Senha confere!")
    else:
        print("Senha NÃO confere.", file=sys.stderr)
        sys.exit(1)


def main():
    parser = argparse.ArgumentParser(
        description="Hashpass - Ferramenta para hashing de senhas"
    )
    parser.add_argument(
        "--algoritmo", "-a",
        default="sha256",
        choices=["sha256", "sha512", "sha3_256", "sha3_512"],
        help="Algoritmo de hash (padrão: sha256)"
    )

    subparsers = parser.add_subparsers(dest="comando", required=True)

    # Subcomando: hash
    hash_parser = subparsers.add_parser("hash", help="Gera hash de uma senha")
    hash_parser.add_argument("senha", help="Senha para hashear")

    # Subcomando: verify
    verify_parser = subparsers.add_parser(
        "verify", help="Verifica senha contra um hash"
    )
    verify_parser.add_argument("senha", help="Senha para verificar")
    verify_parser.add_argument("hash", help="Hash armazenado")
    verify_parser.add_argument("--salt", "-s", required=True,
                               help="Salt usado no hash original")

    args = parser.parse_args()

    if args.comando == "hash":
        cmd_hash(args)
    elif args.comando == "verify":
        cmd_verify(args)


if __name__ == "__main__":
    main()
```

## Como usar

Gerar o hash de uma senha:

```bash
python hashpass.py hash "minhaSenha123"
```

Saída:

```
Salt:      a3f8b2c1d4e5f6a7b8c9d0e1f2a3b4c5...
Hash:      9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
Algoritmo: sha256
```

Verificar se uma senha confere:

```bash
python hashpass.py verify "minhaSenha123" "9f86d081884c7d659a2..." \
    --salt "a3f8b2c1d4e5f6a7b8c9..."
```

## Como funciona

1. **Salt aleatório**: usamos `secrets.token_hex()` para gerar um salt criptograficamente seguro. O módulo `secrets` foi introduzido no Python 3.6 e é preferível ao `random` para fins de segurança.

2. **Concatenação salt + senha**: juntamos o salt à senha antes de hashear. Isso garante que duas senhas iguais gerem hashes diferentes.

3. **Hashing**: usamos `hashlib.new()` para criar o hash com o algoritmo escolhido.

4. **Verificação**: para verificar, basta hashear a senha com o mesmo salt e comparar os resultados.

## Limitações importantes

Para aplicações em produção, considere usar **bcrypt** ou **argon2** em vez de hashlib puro. Essas bibliotecas são projetadas especificamente para senhas e incluem fator de custo configurável:

```python
import bcrypt

# Hash
hashed = bcrypt.hashpw(b"minhaSenha", bcrypt.gensalt(rounds=12))

# Verify
bcrypt.checkpw(b"minhaSenha", hashed)
```

A diferença é que bcrypt é **intencionalmente lento**, o que dificulta ataques de força bruta mesmo com hardware poderoso. O hashlib puro é rápido demais para esse propósito.

## Conclusão

Essa ferramenta demonstra na prática os conceitos de hashing e salting que discuti no artigo anterior. Use-a para aprender, mas em produção prefira bcrypt ou argon2. O código completo está disponível para quem quiser adaptar e melhorar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
