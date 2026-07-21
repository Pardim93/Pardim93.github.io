---
layout: post
author: Wellington Pardim
title:  "Generics no Go 1.18: escrevendo código reutilizável"
categories: "go"
position: 29
---

Go 1.18 trouxe a feature mais pedida da história da linguagem: **Generics** (tipos paramétricos). Agora é possível escrever funções e tipos que trabalham com múltiplos tipos sem sacrificar a segurança de tipos.

## O problema antes dos Generics

Antes do 1.18, se você quisesse uma função `Contains` para slices, precisava de uma para cada tipo:

```python
func ContainsInt(s []int, v int) bool {
    for _, item := range s {
        if item == v {
            return true
        }
    }
    return false
}

func ContainsString(s []string, v string) bool {
    for _, item := range s {
        if item == v {
            return true
        }
    }
    return false
}
```

Ou usar `interface{}` (agora `any`) e perder segurança de tipos:

```python
func Contains(s []any, v any) bool {
    // Sem type safety, precisa de type assertion
}
```

## Generics: a solução

```go
func Contains[T comparable](s []T, v T) bool {
    for _, item := range s {
        if item == v {
            return true
        }
    }
    return false
}

// Uso
Contains([]int{1, 2, 3}, 2)          // true
Contains([]string{"a", "b"}, "c")    // false
```

O `[T comparable]` é um **type parameter** — `T` é o nome do tipo genérico, `comparable` é a **constraint** (restrição).

## Constraints (restrições)

Constraints definem quais tipos são aceitos:

### `any` (alias para `interface{}`)

```go
func Imprime[T any](valor T) {
    fmt.Println(valor)
}
```

### `comparable`

Tipos que suportam `==` e `!=`:

```go
func Igual[T comparable](a, b T) bool {
    return a == b
}
```

### Constraints customizadas com interfaces

```go
type Numero interface {
    ~int | ~int32 | ~int64 | ~float32 | ~float64
}

func Soma[T Numero](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}

Soma([]int{1, 2, 3})       // 6
Soma([]float64{1.5, 2.5})  // 4.0
```

O `~int` significa "qualquer tipo cujo tipo subjacente é int" (permite type aliases).

### `constraints` package

```go
import "golang.org/x/exp/constraints"

func Max[T constraints.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

Max(3, 5)          // 5
Max("abc", "xyz")  // "xyz"
```

## Tipos genéricos

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

// Uso
stack := &Stack[string]{}
stack.Push("a")
stack.Push("b")
val, ok := stack.Pop() // "b", true
```

## Map, Filter, Reduce

```go
func Map[T any, R any](s []T, f func(T) R) []R {
    result := make([]R, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

func Filter[T any](s []T, f func(T) bool) []T {
    var result []T
    for _, v := range s {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}

func Reduce[T any, R any](s []T, init R, f func(R, T) R) R {
    acc := init
    for _, v := range s {
        acc = f(acc, v)
    }
    return acc
}

// Uso
nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

pares := Filter(nums, func(n int) bool { return n%2 == 0 })
dobros := Map(pares, func(n int) int { return n * 2 })
soma := Reduce(dobros, 0, func(acc, n int) int { return acc + n })
// soma = 60
```

## Sets com Generics

```go
type Set[T comparable] map[T]struct{}

func NewSet[T comparable](items ...T) Set[T] {
    s := make(Set[T])
    for _, item := range items {
        s[item] = struct{}{}
    }
    return s
}

func (s Set[T]) Add(item T) {
    s[item] = struct{}{}
}

func (s Set[T]) Contains(item T) bool {
    _, ok := s[item]
    return ok
}

func (s Set[T]) Union(other Set[T]) Set[T] {
    result := NewSet[T]()
    for item := range s {
        result.Add(item)
    }
    for item := range other {
        result.Add(item)
    }
    return result
}

// Uso
a := NewSet(1, 2, 3)
b := NewSet(3, 4, 5)
c := a.Union(b) // {1, 2, 3, 4, 5}
```

## Quando usar Generics

**Use quando:**
- Funções que trabalham com slices/maps de qualquer tipo
- Estruturas de dados genéricas (Stack, Queue, Set, Tree)
- Funções utilitárias (Map, Filter, Reduce)

**Não use quando:**
- Uma interface simples resolve o problema
- O código é específico para um tipo
- A legibilidade sofre

## Conclusão

Generics é a feature que Go precisava para eliminar o código duplicado sem sacrificar a simplicidade. A sintaxe é limpa, as constraints são expressivas, e a performance é a mesma de código não-genérico (Go monomorphiza os tipos em tempo de compilação).

Comece usando generics em funções utilitárias (Contains, Map, Filter) e estruturas de dados. Evite generics excessivos — a filosofia do Go continua sendo "aclaridade é melhor que cleverness".

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
