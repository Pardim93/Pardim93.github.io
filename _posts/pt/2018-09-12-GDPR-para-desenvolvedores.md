---
layout: post
author: Wellington Pardim
title:  "GDPR para desenvolvedores: o que você precisa saber"
categories: "security"
position: 14
---

O **GDPR** (General Data Protection Regulation) entrou em vigor em maio de 2018 na União Europeia e mudou a forma como empresas no mundo inteiro lidam com dados pessoais. Mesmo se você desenvolve software no Brasil, é provável que o GDPR se aplique a você.

## O que é o GDPR?

O GDPR é uma regulamentação europeia que protege dados pessoais dos cidadãos da UE. Ele se aplica a **qualquer empresa que processe dados de residentes europeus**, independentemente de onde a empresa esteja localizada.

Se o seu site ou app tem usuários na Europa, o GDPR se aplica a você.

## Por que o desenvolvedor deve se importar?

O GDPR não é apenas uma questão jurídica — ele tem implicações técnicas diretas no código que você escreve:

- Como você coleta dados?
- Onde você armazena?
- Quem tem acesso?
- Como o usuário pode deletar seus dados?
- Como você notifica sobre violações?

Multas podem chegar a **€20 milhões ou 4% do faturamento global**, o que for maior.

## Princípios do GDPR

### 1. Consentimento explícito

O usuário deve **concordar ativamente** com a coleta de dados. Checkboxes pré-marcadas não são válidas.

```html
<!-- ERRADO: checkbox pré-marcada -->
<input type="checkbox" checked> Aceito os termos

<!-- CORRETO: checkbox vazio -->
<input type="checkbox" name="consent" required>
Concordo com a política de privacidade
```

### 2. Minimização de dados

Colete apenas o que é necessário. Se você precisa do e-mail, não peça CPF:

```python
# ERRADO: coletando dados desnecessários
def registrar_usuario(dados):
    return {
        "nome": dados["nome"],
        "email": dados["email"],
        "cpf": dados["cpf"],        # não é necessário para cadastro
        "data_nascimento": dados["data_nascimento"],  # não é necessário
    }

# CORRETO: apenas o essencial
def registrar_usuario(dados):
    return {
        "nome": dados["nome"],
        "email": dados["email"],
    }
```

### 3. Direito ao esquecimento

O usuário pode pedir a exclusão de todos os seus dados. Você precisa ter um processo para isso:

```python
def deletar_usuario(usuario_id):
    """Remove todos os dados do usuário do sistema."""
    # Deletar dados principais
    db.usuarios.delete_one({"_id": usuario_id})
    
    # Deletar dados derivados
    db.logs.delete_many({"usuario_id": usuario_id})
    db.preferencias.delete_one({"usuario_id": usuario_id})
    db.sessoes.delete_many({"usuario_id": usuario_id})
    
    # Deletar arquivos
    storage.deletar_arquivos(f"usuarios/{usuario_id}/")
    
    # Registrar a exclusão (sem dados pessoais)
    db.auditoria.insert_one({
        "acao": "exclusao_usuario",
        "usuario_id": usuario_id,
        "data": datetime.utcnow(),
    })
```

### 4. Portabilidade de dados

O usuário pode solicitar uma cópia de todos os seus dados em formato legível:

```python
import json
from datetime import datetime

def exportar_dados_usuario(usuario_id):
    """Exporta todos os dados do usuário em formato JSON."""
    usuario = db.usuarios.find_one({"_id": usuario_id})
    pedidos = list(db.pedidos.find({"usuario_id": usuario_id}))
    preferencias = db.preferencias.find_one({"usuario_id": usuario_id})
    
    exportacao = {
        "usuario": {
            "nome": usuario["nome"],
            "email": usuario["email"],
            "data_cadastro": usuario["data_cadastro"].isoformat(),
        },
        "pedidos": [
            {
                "data": p["data"].isoformat(),
                "valor": p["valor"],
                "itens": p["itens"],
            }
            for p in pedidos
        ],
        "preferencias": preferencias,
        "exportado_em": datetime.utcnow().isoformat(),
    }
    
    return json.dumps(exportacao, ensure_ascii=False, indent=2)
```

### 5. Notificação de violações

Se houver um breach de dados, você tem **72 horas** para notificar as autoridades e os afetados. Isso exige monitoramento e logs:

```python
import logging
from datetime import datetime

logger = logging.getLogger("seguranca")

def registrar_tentativa_acesso(usuario_id, resultado, ip):
    """Registra tentativas de acesso para auditoria."""
    logger.info({
        "evento": "tentativa_acesso",
        "usuario_id": usuario_id,
        "resultado": resultado,  # "sucesso" ou "falha"
        "ip": ip,
        "timestamp": datetime.utcnow().isoformat(),
    })

def detectar_breach(tentativas_falha, janela_minutos=5, limite=100):
    """Detecta possível breach baseado em tentativas de acesso."""
    from collections import Counter
    
    ips_suspeitos = Counter()
    for tentativa in tentativas_falha:
        ips_suspeitos[tentativa["ip"]] += 1
    
    suspeitos = [
        ip for ip, count in ips_suspeitos.items() 
        if count > limite
    ]
    
    if suspeitos:
        logger.critical({
            "evento": "possivel_breach",
            "ips_suspeitos": suspeitos,
            "timestamp": datetime.utcnow().isoformat(),
        })
        return True
    return False
``## Checklist para desenvolvedores

Implemente isso no seu projeto:

- [ ] **Consentimento**: checkbox explícito antes de coletar dados
- [ ] **Política de privacidade**: página acessível com linguagem clara
- [ ] **Endpoint de exclusão**: `DELETE /users/me` que remove todos os dados
- [ ] **Endpoint de exportação**: `GET /users/me/data` que retorna JSON
- [ ] **Logs de auditoria**: registre quem acessou o quê e quando
- [ ] **Criptografia**: dados em trânsito (HTTPS) e em repouso (bcrypt para senhas)
- [ ] **Retenção de dados**: delete dados antigos automaticamente
- [ ] **Anonimização**: em logs e analytics, remova identificadores pessoais

## GDPR no contexto brasileiro

O Brasil tem a **LGPD** (Lei Geral de Proteção de Dados), muito inspirada no GDPR. Se você cumpre o GDPR, já está cumprindo boa parte da LGPD. As diferenças principais:

| Aspecto | GDPR | LGPD |
|---------|------|------|
| Vigência | Maio 2018 | Setembro 2020 |
| Multa máxima | €20M ou 4% faturamento | 2% faturamento (até R$50M) |
| Autoridade | DPA de cada país | ANPD |
| Notificação de breach | 72 horas | Prazo razoável |

## Conclusão

O GDPR mudou a forma como pensamos sobre dados de usuários. Como desenvolvedor, pense em privacidade desde o início do projeto — não é algo que se adiciona depois.

Implementar as práticas acima não apenas te protege legalmente, mas também demonstra respeito pelos seus usuários. Em um mundo onde dados valem ouro, tratar bem os dados dos seus usuários é um diferencial competitivo.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
