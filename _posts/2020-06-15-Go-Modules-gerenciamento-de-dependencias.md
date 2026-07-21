---
layout: post
author: Wellington Pardim
title:  "Go Modules: gerenciamento de dependências no Go"
categories: "go"
position: 21
---

Antes do Go Modules, gerenciar dependências em Go era confuso — `GOPATH`, `dep`, `glide`, `govendor`... Cada um tinha sua ferramenta. Com o Go Modules (introduzido no Go 1.11 e estabilizado no 1.13), finalmente temos uma solução oficial e integrada à linguagem.

## O que é Go Modules?

Go Modules é o sistema oficial de gerenciamento de dependências do Go. Ele controla:

- **Quais** dependências seu projeto usa
- **Quais versões** de cada dependência
- **Integridade** das dependências (hashes)

O arquivo `go.mod` é o coração do sistema — equivalente ao `requirements.txt` do Python ou `package.json` do Node.

## Inicializando um módulo

```bash
mkdir meu-projeto && cd meu-projeto
go mod init github.com/Pardim93/meu-projeto
```

Isso cria o `go.mod`:

```
module github.com/Pardim93/meu-projeto

go 1.21
```

## Adicionando dependências

Quando você importa um package e roda `go build`, o Go automaticamente adiciona a dependência:

```go
package main

import (
    "fmt"
    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()
    fmt.Println("Router criado!")
}
```

```bash
go build
# O Go baixa gorilla/mux automaticamente
```

O `go.mod` é atualizado:

```
module github.com/Pardim93/meu-projeto

go 1.21

require github.com/gorilla/mux v1.8.0
```

E o `go.sum` é criado com hashes de integridade.

## Comandos essenciais

```bash
# Adicionar uma dependência específica
go get github.com/gorilla/mux@v1.8.0

# Atualizar para última versão
go get -u github.com/gorilla/mux

# Atualizar todas as dependências
go get -u ./...

# Remover dependências não usadas
go mod tidy

# Verificar integridade
go mod verify

# Baixar dependências para cache local
go mod download

# Listar dependências
go list -m all

# Ver o porquê de uma dependência existir
go mod why github.com/gorilla/mux
```

## Versionamento semântico

Go Modules usa **semver** (Semantic Versioning):

```
v1.2.3
│ │ │
│ │ └── Patch: correções de bugs
│ └──── Minor: novas features (backward compatible)
└────── Major: breaking changes
```

No `go.mod`:

```
require (
    github.com/lib/pq v1.10.4          // versão exata
    github.com/redis/go-redis/v9 v9.0.2 // v9 é um módulo diferente
)
```

## Major versions

Se um package muda a API (breaking change), ele muda o **caminho do módulo**:

```
// v0 e v1: mesmo caminho
github.com/foo/bar

// v2+: caminho diferente
github.com/foo/bar/v2
```

No seu código:

```go
import "github.com/foo/bar/v2"
```

## Replace directives

Útil para desenvolvimento local ou forks:

```go
// go.mod
module github.com/meu/projeto

require github.com/dependencia/v2 v2.1.0

// Apontar para versão local durante desenvolvimento
replace github.com/dependencia/v2 => ../dependencia

// Apontar para um fork
replace github.com/dependencia/v2 => github.com/meu-fork/dependencia/v2 v2.1.1
```

## Vendor mode

Para ambientes onde não é possível baixar dependências (CI restrito, air-gapped environments):

```bash
# Copia todas as dependências para ./vendor
go mod vendor

# Build usando vendor
go build -mod=vendor

# Atualizar vendor após mudanças no go.mod
go mod vendor
```

O diretório `vendor/` deve ser commitado nesses casos.

## `go.sum`: o que é?

O `go.sum` contém hashes criptográficos de cada versão de dependência:

```
github.com/gorilla/mux v1.8.0 h1:...
github.com/gorilla/mux v1.8.0/go.mod h1:...
```

Ele garante que ninguém adulterou o código após você ter baixado. **Sempre commit o `go.sum`**.

## Dicas práticas

**1. Sempre rode `go mod tidy` antes de commit:**

```bash
go mod tidy
git add go.mod go.sum
git commit -m "Update dependencies"
```

**2. Use `go mod graph` para visualizar dependências:**

```bash
go mod graph | head -20
```

**3. Use `go mod edit` para edições programáticas:**

```bash
# Adicionar um replace
go mod edit -replace github.com/old/pkg=github.com/new/pkg@v1.0.0

# Setar a versão Go
go mod edit -go=1.21
```

**4. Cache global:**

O Go mantém um cache global em `~/go/pkg/mod`. Múltiplos projetos compartilham o mesmo cache, economizando espaço e tempo.

## Comparação com outras linguagens

| Feature | Go Modules | pip (Python) | npm (Node) |
|---------|-----------|-------------|------------|
| Arquivo de config | go.mod | requirements.txt | package.json |
| Lock file | go.sum | requirements.txt (pinado) | package-lock.json |
| Gerenciado pelo build | Sim | Não | Não |
| Cache global | Sim | Sim (~/.cache/pip) | Sim (~/.npm) |
| Vendor mode | Sim (go mod vendor) | Não nativo | Sim (node_modules) |

## Conclusão

Go Modules resolveu definitivamente o problema de dependências no Go. É simples, integrado ao toolchain, e "apenas funciona". Se você ainda está usando `dep` ou `GOPATH`, migre — não há razão para não usar.

O fluxo de trabalho básico é: `go mod init`, importe os packages, `go mod tidy`, commit. O Go cuida do resto.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
