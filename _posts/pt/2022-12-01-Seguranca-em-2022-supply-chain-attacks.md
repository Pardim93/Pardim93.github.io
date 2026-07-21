---
layout: post
author: Wellington Pardim
title:  "Segurança em 2022: supply chain attacks e como se defender"
categories: "security"
position: 31
---

Em 2022, ataques à cadeia de suprimentos se consolidaram como uma das principais ameaças. De pacotes npm comprometidos a vulnerabilidades em bibliotecas usadas por milhões, a superfície de ataque cresceu exponencialmente. Vamos entender as ameaças e como se proteger.

## O cenário atual

Em 2022, vimos:

- **node-ipc**: maintainer sabotou propositalmente seu próprio pacote (protesto político), afetando milhões de downloads
- **colors e faker**: mesmo autor, mesmo sabotou os pacotes
- **Log4Shell (final 2021)**: continuou causando estragos em 2022
- **Spring4Shell**: vulnerabilidade no Spring Framework
- **Ataques a GitHub Actions**: workflows mal configurados explorados

O padrão é claro: **atacantes não miram mais apenas seu código — miram suas dependências**.

## Tipos de supply chain attacks

### 1. Comprometimento de mantenedor

O atacante rouba credenciais de um maintainer de pacote popular:

```
Atacante → Rouba npm token do maintainer
         → Publica versão maliciosa do pacote
         → Milhares de projetos atualizam automaticamente
```

### 2. Typosquatting

Criação de pacotes com nomes similares a populares:

```bash
# Pacotes reais
npm install lodash
npm install requests

# Pacotes maliciosos (typosquatting)
npm install l0dash      # zero em vez de 'o'
npm install requestss   # 's' extra
npm install requsts     # 'e' faltando
```

### 3. Dependency confusion

O atacante publica um pacote com o mesmo nome de um pacote interno da empresa, mas no registry público. O gerenciador de pacotes pode baixar o público em vez do interno.

### 4. Comprometimento de build

Como no SolarWinds, o atacante compromete o pipeline de CI/CD:

```
Código fonte limpo → Build comprometido → Binário malicioso
```

## Como se defender

### 1. Lock files sempre

```bash
# Python: pinne versões
requests==2.28.0

# Node: package-lock.json deve ser commitado
git add package-lock.json

# Go: go.sum já faz isso
git add go.sum
```

### 2. Verifique integridade

```bash
# npm: verifique hashes
npm audit

# pip: verifique vulnerabilidades
pip audit

# Go: verifique integridade
go mod verify

# Verifique hashes de downloads
sha256sum -c checksums.txt
```

### 3. Menos dependências

Cada dependência é uma superfície de ataque:

```python
# Pergunte-se antes de adicionar:
# 1. Eu realmente preciso disso?
# 2. O maintainer é confiável?
# 3. Quando foi a última atualização?
# 4. Quantos downloads tem?
# 5. Tem vulnerabilidades conhecidas?
```

### 4. SBOM (Software Bill of Materials)

Mantenha um inventário de todas as dependências:

```bash
# Python
pip install pip-audit
pip-audit

# Use Syft para gerar SBOM completo
syft dir:. -o spdx-json > sbom.json
```

### 5. GitHub Actions seguras

```yaml
# ERRADO: usar tag mutable
- uses: actions/checkout@main

# CORRETO: usar hash de commit
- uses: actions/checkout@93ea575cb5d8a053eaa0ac8fa3b40d7e05a33cc8 # v3.1.0

# ERRADO: permissões amplas
permissions: write-all

# CORRETO: permissões mínimas
permissions:
  contents: read
  pull-requests: write
```

### 6. Dependabot / Renovate

Configure updates automáticos com CI:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

### 7. Code signing

Assine seus releases:

```bash
# GPG signing de commits
git commit -S -m "feat: new feature"

# Sigstore para containers
cosign sign --key cosign.key minha-imagem:latest
```

### 8. Network policies

Em produção, restrinja egress:

```yaml
# Kubernetes NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/8  # Apenas rede interna
```

## Ferramentas de segurança

| Ferramenta | O que faz |
|-----------|----------|
| `npm audit` | Vulnerabilidades em dependências npm |
| `pip audit` | Vulnerabilidades em dependências Python |
| `govulncheck` | Vulnerabilidades em dependências Go |
| `Snyk` | Scan de vulnerabilidades (multi-linguagem) |
| `Trivy` | Scan de containers e IaC |
| `Sigstore` | Code signing sem certificados |
| `Scorecard` | Avalia segurança de repositórios GitHub |

## Checklist de segurança para projetos

- [ ] Lock files commitados
- [ ] Dependabot/Renovate configurado
- [ ] CI roda `npm audit` / `pip audit` em cada PR
- [ ] GitHub Actions usam hashes, não tags
- [ ] Permissions mínimas em workflows
- [ ] Secrets não hardcoded (use variáveis de ambiente)
- [ ] Code signing em releases
- [ ] SBOM gerado e publicado
- [ ] Review de PRs de dependências externas

## Conclusão

Supply chain security não é mais opcional — é obrigatório. Cada dependência que você adiciona é uma porta potencial para atacantes. A defesa é em camadas: lock files, auditoria, updates, code signing, e principalmente — pensamento crítico antes de adicionar uma nova dependência.

Comece pelo básico: rode `npm audit` ou `pip audit` nos seus projetos hoje. Você vai se surpreender com o que encontrar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
