---
layout: post
author: Wellington Pardim
title:  "Ataque SolarWinds: anatomy de um supply chain attack"
categories: "security"
position: 22
---

Em dezembro de 2020, o mundo descobriu um dos ataques cibernéticos mais sofisticados da história. O **SolarWinds** (apelidado de SUNBURST) comprometeu a cadeia de suprimentos de software, afetando empresas como Microsoft, Intel, Cisco e agências do governo dos EUA. Vamos entender como funcionou.

## O que é um supply chain attack?

Em vez de atacar diretamente o alvo, o atacante compromete uma **dependência** que o alvo usa. É como envenenar o suprimento de água em vez de envenenar cada poço individualmente.

```
Atacante → Compromete Software A → Software A é usado por B, C, D...
         → B, C, D são infectados sem saber
```

## Timeline

- **Outubro 2019**: Atacantes testam o código malicioso em ambiente de testes da SolarWinds
- **Março 2020**: Código malicioso é inserido no build do **Orion**, software de monitoramento de rede da SolarWinds
- **Março-Dezembro 2020**: ~18.000 organizações instalam a versão comprometida
- **Dezembro 2020**: FireEye (empresa de segurança) descobre o ataque após seu próprio comprometimento
- **Dezembro 2020**: SolarWinds emite patch e notifica clientes

## Como funcionou

### 1. Comprometimento do build

Os atacantes (atribuídos ao grupo APT29/Cozy Bear, ligado à Rússia) conseguiram acesso ao **pipeline de build** da SolarWinds. Eles não alteraram o código fonte diretamente — inseriram código malicioso **durante o processo de compilação**.

O código foi adicionado ao arquivo `SolarWinds.Orion.Core.BusinessLayer.dll`:

```
Orion Improvement Business Layer (OIBL)
├── Código legítimo do Orion
├── ← Código malicioso inserido aqui
└── Mais código legítimo
```

### 2. Backdoor (SUNBURST)

O código malicioso era uma backdoor chamada **SUNBURST** que:

1. **Dormia por 2 semanas** após instalação (evita detecção em sandbox)
2. **Verificava por software de segurança** — se encontrasse, não executava
3. **Comunicava com C2** (Command & Control) via DNS:
   - Gerava subdomínios únicos codificados em base32
   - Fazia queries DNS para `avsvmcloud.com`
   - O servidor C2 respondia com IPs nos ranges da SolarWinds

```
Exemplo de comunicação:
[encoded_machine_info].avsvmcloud.com
     ↓
Resposta DNS: IP no range 10.174.x.x (sinal de "continue")
ou
Resposta DNS: IP externo (sinal de "execute comando")
```

### 3. Movimentação lateral

Uma vez dentro da rede, os atacantes:

- Roubavam credenciais de administradores
- Acessavam servidores de e-mail (On-premise Exchange via ADFS)
- Criavam tokens SAML falsos para acessar recursos cloud (Microsoft 365, Azure)
- Exfiltravam dados sensíveis

### 4. Anti-forense

Os atacantes eram meticulosos:

- Usavam IPs residenciais (VPN) para não levantar suspeitas
- Limpavam logs após cada sessão
- Não deixavam malware permanente — operavam "living off the land"
- Desativavam o backdoor se detectassem monitoramento

## Por que foi tão grave

1. **Escala**: 18.000 organizações instalaram o software comprometido
2. **Alvos de alto valor**: Microsoft, Departamento de Tesouro dos EUA, Departamento de Energia, NATO
3. **Período longo**: 9 meses de acesso não detectado
4. **Confiança na cadeia**: as vítimas confiavam na SolarWinds — era um fornecedor legítimo
5. **Dificuldade de detecção**: o código malicioso era Ofuscado e se misturava ao código legítimo

## Lições para desenvolvedores

### 1. Verifique suas dependências

```bash
# Python: verifique vulnerabilidades conhecidas
pip audit

# Node: verifique vulnerabilidades
npm audit

# Go: verifique hashes
go mod verify
```

### 2. Use lock files

Lock files garantem que você instale exatamente as mesmas versões:

```python
# Python: pinne versões
requests==2.28.0  # não requests>=2.0
```

```bash
# Go: go.sum já faz isso
# Node: package-lock.json
```

### 3. Code signing

Assine seus builds para garantir integridade:

```bash
# Gerar checksum do build
sha256sum meu-app > meu-app.sha256

# Verificar
sha256sum -c meu-app.sha256
```

### 4. Menor privilégio para CI/CD

Seu pipeline de build não deve ter acesso a tudo:

```yaml
# GitHub Actions: use secrets com escopo mínimo
- uses: aws-actions/configure-aws-credentials@v2
  with:
    role-to-assume: arn:aws:iam::123456:role/build-role
    # Não use credenciais de admin!
```

### 5. SBOM (Software Bill of Materials)

Mantenha um inventário de todas as dependências:

```bash
# Python
pip freeze > requirements.txt

# Go
go mod graph > dependencies.txt

# Use ferramentas como Syft para gerar SBOM completo
```

### 6. Monitore anomalias

- Tráfego DNS incomum (como queries para domínios gerados aleatoriamente)
- Acessos fora de horário comercial
- Múltiplas tentativas de login falhadas
- Exfiltração de dados em volumes anormais

## Conexão com outros ataques

O SolarWinds não foi o primeiro supply chain attack:

- **event-stream (2018)**: pacote npm comprometido para roubar Bitcoin
- **Codecov (2021)**: script de upload de cobertura modificado para roubar variáveis de ambiente
- **Log4Shell (2021)**: vulnerabilidade em biblioteca usada por milhões
- **XZ Utils (2024)**: backdoor inserido em biblioteca de compressão Linux

A tendência é clara: ataques à cadeia de suprimentos estão aumentando.

## Conclusão

O ataque SolarWinds mudou a forma como pensamos sobre segurança de software. Não basta proteger seu próprio código — você precisa proteger toda a cadeia de dependências, desde o código fonte até o binário final.

Como desenvolvedor, comece com o básico: pinne versões, verifique hashes, use SBOMs e monitore anomalias. A segurança é tão forte quanto o elo mais fraco da cadeia.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
