---
layout: post
author: Wellington Pardim
title:  "Passkeys e autenticação sem senha: o futuro é agora"
categories: "security"
position: 37
---

Senhas são o elo mais fraco da segurança. **81% dos breaches** envolvem credenciais comprometidas. Os **Passkeys** são a alternativa que a indústria está adotando: autenticação sem senha, resistente a phishing, e mais fácil de usar.

## O que são Passkeys?

Passkeys usam **criptografia assimétrica** (chave pública/privada) para autenticação:

1. **Registro**: o dispositivo gera um par de chaves. A chave privada fica no dispositivo, a pública vai para o servidor.
2. **Autenticação**: o servidor envia um desafio. O dispositivo assina com a chave privada. O servidor verifica com a chave pública.

```
Usuário → Servidor: "Quero logar"
Servidor → Usuário: "Assine este desafio"
Usuário → Servidor: [assinatura com chave privada]
Servidor: Verifica com chave pública ✓
```

## Por que é mais seguro

| Aspecto | Senhas | Passkeys |
|---------|--------|----------|
| Phishing | Vulnerável | Imune (verifica origem) |
| Reutilização | Comum | Impossível (chave por serviço) |
| Brute force | Possível | Impossível (chave de 256 bits) |
| Keylogger | Captura senha | Nada para capturar |
| Storage no servidor | Hash (pode ser quebrado) | Apenas chave pública |

## Implementando com WebAuthn

### Backend (Python/Flask)

```python
from flask import Flask, request, jsonify, session
from webauthn import (
    generate_registration_options,
    verify_registration_response,
    generate_authentication_options,
    verify_authentication_response,
)
from webauthn.helpers.structs import (
    AuthenticatorSelectionCriteria,
    ResidentKeyRequirement,
)
import json

app = Flask(__name__)
app.secret_key = "sua-chave-secreta"

# Armazenamento (use banco de dados em produção)
usuarios = {}
credenciais = {}

RP_NAME = "Meu Site"
RP_ID = "localhost"
ORIGIN = "http://localhost:5000"

@app.route("/register/options", methods=["POST"])
def register_options():
    usuario = request.json["username"]
    
    options = generate_registration_options(
        rp_name=RP_NAME,
        rp_id=RP_ID,
        user_name=usuario,
        user_id=usuario.encode(),
        authenticator_selection=AuthenticatorSelectionCriteria(
            resident_key=ResidentKeyRequirement.REQUIRED,
        ),
    )
    
    session["challenge"] = options.challenge
    session["username"] = usuario
    
    return jsonify(json.loads(options.to_json()))

@app.route("/register/verify", methods=["POST"])
def register_verify():
    try:
        verification = verify_registration_response(
            credential=request.json,
            expected_challenge=session["challenge"],
            expected_origin=ORIGIN,
            expected_rp_id=RP_ID,
        )
        
        usuario = session["username"]
        credenciais[usuario] = {
            "credential_id": verification.credential_id,
            "public_key": verification.credential_public_key,
            "sign_count": verification.sign_count,
        }
        
        return jsonify({"verified": True})
    except Exception as e:
        return jsonify({"verified": False, "error": str(e)}), 400

@app.route("/login/options", methods=["POST"])
def login_options():
    usuario = request.json["username"]
    
    if usuario not in credenciais:
        return jsonify({"error": "Usuário não encontrado"}), 404
    
    options = generate_authentication_options(
        rp_id=RP_ID,
        allow_credentials=[{
            "type": "public-key",
            "id": credenciais[usuario]["credential_id"],
        }],
    )
    
    session["challenge"] = options.challenge
    session["username"] = usuario
    
    return jsonify(json.loads(options.to_json()))

@app.route("/login/verify", methods=["POST"])
def login_verify():
    try:
        usuario = session["username"]
        cred = credenciais[usuario]
        
        verification = verify_authentication_response(
            credential=request.json,
            expected_challenge=session["challenge"],
            expected_origin=ORIGIN,
            expected_rp_id=RP_ID,
            credential_public_key=cred["public_key"],
            credential_current_sign_count=cred["sign_count"],
        )
        
        cred["sign_count"] = verification.new_sign_count
        session["autenticado"] = True
        
        return jsonify({"verified": True})
    except Exception as e:
        return jsonify({"verified": False, "error": str(e)}), 400
```

### Frontend (JavaScript)

```javascript
// Registrar passkey
async function registrar(username) {
  const options = await fetch("/register/options", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username }),
  }).then(r => r.json());

  const credential = await navigator.credentials.create({
    publicKey: options,
  });

  const response = await fetch("/register/verify", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(credential),
  }).then(r => r.json());

  return response.verified;
}

// Login com passkey
async function login(username) {
  const options = await fetch("/login/options", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username }),
  }).then(r => r.json());

  const assertion = await navigator.credentials.get({
    publicKey: options,
  });

  const response = await fetch("/login/verify", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(assertion),
  }).then(r => r.json());

  return response.verified;
}
```

## Synced Passkeys

O grande diferencial dos passkeys modernos é o **sync**:

- **Apple**: passkeys sincronizam via iCloud Keychain
- **Google**: passkeys sincronizam via Google Password Manager
- **Microsoft**: passkeys sincronizam via Windows Hello

Isso significa que um passkey criado no iPhone funciona automaticamente no MacBook. Não há risco de perder o dispositivo.

## Adoção

As principais empresas já suportam passkeys:

- **Apple**: desde iOS 16 / macOS Ventura
- **Google**: desde Android 14 / Chrome
- **Microsoft**: desde Windows 11
- **GitHub**: suporta passkeys desde 2023
- **1Password**: armazena e sincroniza passkeys

## Limitações atuais

- **Suporte do browser**: ainda não universal (mas perto)
- **Device-bound vs synced**: alguns passkeys ficam presos ao dispositivo
- **Recovery**: se perder todos os dispositivos, precisa de mecanismo de recuperação
- **Usuários menos técnicos**: curva de aprendizado

## Conclusão

Passkeys representam a maior mudança em autenticação desde a invenção da senha. São mais seguros (imunes a phishing), mais fáceis (sem senhas para lembrar), e a adoção está crescendo exponencialmente.

Se você desenvolve aplicações web, comece a implementar suporte a passkeys hoje. A biblioteca `webauthn` para Python torna o processo relativamente simples. Seus usuários vão agradecer.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
