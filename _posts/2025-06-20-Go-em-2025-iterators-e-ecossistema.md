---
layout: post
author: Wellington Pardim
title:  "Go em 2025: iterators, range-over-func e o ecossistema"
categories: "go"
position: 41
---

O Go continua evoluindo sem perder sua essência de simplicidade. As versões recentes trouxeram **iterators**, **range-over-func**, melhorias no ecossistema de ferramentas, e continuidade na adoção de generics.

## Iterators (iter package)

O Go 1.23 introduziu o pacote `iter` (PEP 723) — uma forma padronizada de criar e consumir iteradores:

```go
package main

import (
    "fmt"
    "iter"
)

// Iterator que produz números de 0 a n
func Range(n int) iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := range n {
            if !yield(i) {
                return
            }
        }
    }
}

// Iterator que produz pares (índice, valor)
func Enumerate[T any](s []T) iter.Seq2[int, T] {
    return func(yield func(int, T) bool) {
        for i, v := range s {
            if !yield(i, v) {
                return
            }
        }
    }
}

func main() {
    // Consumindo iterator com range
    for v := range Range(5) {
        fmt.Println(v) // 0, 1, 2, 3, 4
    }

    // Consumindo iterator com 2 valores
    nomes := []string{"Ana", "Bob", "Carlos"}
    for i, nome := range Enumerate(nomes) {
        fmt.Printf("%d: %s\n", i, nome)
    }
}
```

## Range-over-func

A grande feature do Go 1.24: você pode usar `range` com funções:

```go
// Antes: precisava de um objeto com método Next()
// Agora: range sobre uma função

func Fibonacci() iter.Seq[int] {
    return func(yield func(int) bool) {
        a, b := 0, 1
        for {
            if !yield(a) {
                return
            }
            a, b = b, a+b
        }
    }
}

func main() {
    for n := range Fibonacci() {
        if n > 1000 {
            break
        }
        fmt.Println(n) // 0, 1, 1, 2, 3, 5, 8, 13, ...
    }
}
```

## Funcções utilitárias com iterators

```go
func Map[T any, R any](seq iter.Seq[T], f func(T) R) iter.Seq[R] {
    return func(yield func(R) bool) {
        for v := range seq {
            if !yield(f(v)) {
                return
            }
        }
    }
}

func Filter[T any](seq iter.Seq[T], f func(T) bool) iter.Seq[T] {
    return func(yield func(T) bool) {
        for v := range seq {
            if f(v) {
                if !yield(v) {
                    return
                }
            }
        }
    }
}

func Take[T any](seq iter.Seq[T], n int) iter.Seq[T] {
    return func(yield func(T) bool) {
        count := 0
        for v := range seq {
            if count >= n {
                return
            }
            if !yield(v) {
                return
            }
            count++
        }
    }
}

// Uso
func main() {
    nums := Range(100)
    pares := Filter(nums, func(n int) bool { return n%2 == 0 })
    dobrados := Map(pares, func(n int) int { return n * 2 })
    primeiros := Take(dobrados, 5)

    for v := range primeiros {
        fmt.Println(v) // 0, 4, 8, 12, 16
    }
}
```

## Melhorias no toolchain

### `go tool` aprimorado

```bash
# Trace de performance
go tool trace ./...

# Análise de cobertura melhorada
go tool cover -html=coverage.out

# Debug com delve integrado
go tool dlv debug .
```

### `govulncheck` integrado

Verificação de vulnerabilidades agora faz parte do workflow:

```bash
# Instalar
go install golang.org/x/vuln/cmd/govulncheck@latest

# Verificar projeto
govulncheck ./...

# Saída:
# Vulnerability #1: GO-2024-1234
#     Package: golang.org/x/net
#     Fixed in: v0.17.0
```

## Struct tags com `go generate`

```go
//go:generate go run github.com/mailru/easyjson/easyjson -all user.go

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

## Erros melhorados

```go
import "errors"

// errors.Join para múltiplos erros (desde 1.20)
func processarLote(itens []Item) error {
    var errs []error
    for _, item := range itens {
        if err := processar(item); err != nil {
            errs = append(errs, fmt.Errorf("item %d: %w", item.ID, err))
        }
    }
    return errors.Join(errs...)
}

// errors.As com múltiplos tipos
var errValidacao *ValidationError
var errBanco *DatabaseError
if errors.As(err, &errValidacao) {
    // Tratar erro de validação
} else if errors.As(err, &errBanco) {
    // Tratar erro de banco
}
```

## slog: structured logging

```go
import "log/slog"

func main() {
    // JSON logger
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    
    logger.Info("servidor iniciado",
        "porta", 8080,
        "ambiente", "producao",
    )
    
    logger.Error("erro ao processar",
        "erro", err,
        "usuario_id", userID,
    )
}
```

## Ecossistema maduro

| Categoria | Bibliotecas populares |
|-----------|----------------------|
| Web framework | Gin, Echo, Fiber |
| ORM | GORM, sqlx, ent |
| CLI | Cobra, urfave/cli |
| Config | Viper, envconfig |
| Testing | testify, gomock |
| Logging | slog, zerolog, zap |
| HTTP client | resty, net/http |
| Validation | go-playground/validator |

## Conclusão

Go em 2025 é a mesma linguagem simples que conquistou o mundo backend, mas com ferramentas melhores. Iterators e range-over-func trazem functional programming de forma idiomática. Generics eliminam código duplicado. O ecossistema é maduro e confiável.

Se você não experimentou Go recentemente, agora é a hora. A linguagem está no melhor momento de sua história.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
