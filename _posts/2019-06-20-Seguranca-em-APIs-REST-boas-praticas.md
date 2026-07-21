---
layout: post
author: Wellington Pardim
title:  "Segurança em APIs REST: boas práticas com exemplos"
categories: "security python"
position: 17
---

APIs REST são a espinha dorsal da maioria das aplicações modernas. Mas com essa exposição vem responsabilidade — APIs são alvos constantes de ataques. Nesse artigo vou cobrir as principais vulnerabilidades e como se proteger com exemplos em Python.

## 1. Autenticação com JWT

JWT (JSON Web Token) é o método mais comum para autenticar APIs. O fluxo é:

1. Usuário envia credenciais → servidor valida → retorna JWT
2. Usuário envia JWT em cada requisição → servidor verifica

```python
import jwt
from datetime import datetime, timedelta
from functools import wraps
from flask import Flask, request, jsonify

app = Flask(__name__)
CHAVE_SECRETA = "sua-chave-super-secreta"  # Use variável de ambiente!

def gerar_token(usuario_id: int) -> str:
    payload = {
        "sub": usuario_id,
        "iat": datetime.utcnow(),
        "exp": datetime.utcnow() + timedelta(hours=1),
    }
    return jwt.encode(payload, CHAVE_SECRETA, algorithm="HS256")

def verificar_token(token: str) -> dict:
    try:
        return jwt.decode(token, CHAVE_SECRETA, algorithms=["HS256"])
    except jwt.ExpiredSignatureError:
        raise ValueError("Token expirado")
    except jwt.InvalidTokenError:
        raise ValueError("Token inválido")

def autenticado(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return jsonify({"erro": "Token ausente"}), 401
        try:
            dados = verificar_token(token)
            request.usuario_id = dados["sub"]
        except ValueError as e:
            return jsonify({"erro": str(e)}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/login", methods=["POST"])
def login():
    dados = request.get_json()
    # Valide as credenciais contra o banco
    usuario = autenticar(dados["email"], dados["senha"])
    if not usuario:
        return jsonify({"erro": "Credenciais inválidas"}), 401
    return jsonify({"token": gerar_token(usuario["id"])})

@app.route("/perfil")
@autenticado
def perfil():
    return jsonify({"usuario_id": request.usuario_id})
```

## 2. Rate Limiting

Proteja contra brute force e abuso:

```python
from collections import defaultdict
from time import time

class RateLimiter:
    def __init__(self, max_requests=100, janela_segundos=60):
        self.max_requests = max_requests
        self.janela = janela_segundos
        self.requisicoes = defaultdict(list)

    def verificar(self, ip: str) -> bool:
        agora = time()
        # Remove requisições antigas
        self.requisicoes[ip] = [
            t for t in self.requisicoes[ip]
            if agora - t < self.janela
        ]
        if len(self.requisicoes[ip]) >= self.max_requests:
            return False
        self.requisicoes[ip].append(agora)
        return True

limiter = RateLimiter(max_requests=60, janela_segundos=60)

@app.before_request
def verificar_rate_limit():
    ip = request.remote_addr
    if not limiter.verificar(ip):
        return jsonify({"erro": "Rate limit excedido"}), 429
```

## 3. Validação de entrada

**Nunca confie em dados do usuário.** Sempre valide:

```python
from marshmallow import Schema, fields, validate, ValidationError

class CriarUsuarioSchema(Schema):
    nome = fields.Str(required=True, validate=validate.Length(min=2, max=100))
    email = fields.Email(required=True)
    senha = fields.Str(required=True, validate=validate.Length(min=8))
    idade = fields.Int(validate=validate.Range(min=0, max=150))

schema = CriarUsuarioSchema()

@app.route("/usuarios", methods=["POST"])
def criar_usuario():
    try:
        dados = schema.load(request.get_json())
    except ValidationError as err:
        return jsonify({"erros": err.messages}), 400

    # Dados validados e limpos
    usuario = criar_no_banco(dados)
    return jsonify(usuario), 201
```

## 4. CORS (Cross-Origin Resource Sharing)

Se seu frontend roda em outro domínio, configure CORS corretamente:

```python
from flask_cors import CORS

# ERRADO: permite qualquer origem
CORS(app)

# CORRETO: permite apenas origens específicas
CORS(app, origins=["https://meusite.com", "https://admin.meusite.com"])
```

## 5. Sanitização contra SQL Injection

Use ORM ou queries parametrizadas **sempre**:

```python
# ERRADO: SQL injection vulnerável
def buscar_usuario(nome):
    query = f"SELECT * FROM usuarios WHERE nome = '{nome}'"  # PERIGO!

# CORRETO: query parametrizada
def buscar_usuario(nome):
    cursor.execute("SELECT * FROM usuarios WHERE nome = %s", (nome,))
    return cursor.fetchone()

# MELHOR AINDA: usar ORM
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class Usuario(Base):
    __tablename__ = "usuarios"
    id = Column(Integer, primary_key=True)
    nome = Column(String(100))

# SQLAlchemy já protege contra injection
usuario = session.query(Usuario).filter(Usuario.nome == nome).first()
```

## 6. Headers de segurança

```python
@app.after_request
def headers_seguranca(response):
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
    response.headers["Content-Security-Policy"] = "default-src 'self'"
    response.headers["Cache-Control"] = "no-store"
    return response
```

## 7. Logging e monitoramento

Registre eventos de segurança para detecção de ataques:

```python
import logging

logger = logging.getLogger("api.security")

@app.before_request
def log_requisicao():
    logger.info({
        "metodo": request.method,
        "path": request.path,
        "ip": request.remote_addr,
        "user_agent": request.user_agent.string,
    })

@app.after_request
def log_resposta(response):
    if response.status_code >= 400:
        logger.warning({
            "status": response.status_code,
            "path": request.path,
            "ip": request.remote_addr,
        })
    return response
```

## Checklist de segurança para APIs

- [ ] Autenticação em todos os endpoints (exceto login/registro)
- [ ] Rate limiting por IP e por usuário
- [ ] Validação de todos os campos de entrada
- [ ] HTTPS obrigatório em produção
- [ ] CORS configurado para origens específicas
- [ ] Queries parametrizadas (nunca concatenar strings em SQL)
- [ ] Headers de segurança configurados
- [ ] Logs de auditoria para ações sensíveis
- [ ] Senhas hasheadas com bcrypt (nunca MD5 ou SHA)
- [ ] Tokens com expiração curta e refresh tokens

## Conclusão

Segurança em APIs não é opcional — é uma camada que deve estar presente desde o primeiro commit. As práticas acima não são difíceis de implementar e previnem a grande maioria dos ataques comuns.

Comece pelo básico: autenticação JWT, validação de entrada e rate limiting. Depois adicione as camadas extras conforme a aplicação cresce.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
