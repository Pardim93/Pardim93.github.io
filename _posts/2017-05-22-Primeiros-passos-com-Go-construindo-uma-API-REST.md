---
layout: post
author: Wellington Pardim
title:  "Primeiros passos com Go: construindo uma API REST"
categories: "go"
position: 9
---

Go tem ganhado cada vez mais espaço no desenvolvimento backend. Sua simplicidade, performance e suporte nativo a concorrência fazem dele uma escolha excelente para APIs. Nesse artigo vou mostrar como construir uma API REST completa do zero.

## Por que Go?

Go (ou Golang) foi criado no Google em 2009 e se destaca por:

- **Compila para binário nativo** — sem dependência de runtime
- **Concorrência nativa** com goroutines e channels
- **Sintaxe simples** — menos é mais
- **Standard library poderosa** — `net/http` já é suficiente para a maioria dos casos
- **Build rápido** — compila em segundos

## Instalação

No Ubuntu:

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```

No macOS:

```bash
brew install go
```

Verifique:

```bash
go version
```

## Estrutura do projeto

```
api-go/
├── go.mod
├── go.sum
├── main.go
└── handlers/
    └── tasks.go
```

## Inicializando o módulo

```bash
mkdir api-go && cd api-go
go mod init api-go
go get github.com/gorilla/mux
```

## O modelo de dados

Vamos criar uma API de gerenciamento de tarefas. Primeiro, o modelo em `main.go`:

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "strconv"
    "sync"

    "github.com/gorilla/mux"
)

type Task struct {
    ID        int    `json:"id"`
    Title     string `json:"title"`
    Completed bool   `json:"completed"`
}

var (
    tasks  = []Task{}
    nextID = 1
    mu     sync.Mutex
)
```

Note os `json:"..."` tags — eles controlam como os campos são serializados para JSON.

## Handlers

```go
func getTasks(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(tasks)
}

func getTask(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])

    for _, task := range tasks {
        if task.ID == id {
            json.NewEncoder(w).Encode(task)
            return
        }
    }
    http.Error(w, `{"error": "tarefa não encontrada"}`, http.StatusNotFound)
}

func createTask(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    var task Task
    if err := json.NewDecoder(r.Body).Decode(&task); err != nil {
        http.Error(w, `{"error": "JSON inválido"}`, http.StatusBadRequest)
        return
    }

    mu.Lock()
    task.ID = nextID
    nextID++
    tasks = append(tasks, task)
    mu.Unlock()

    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(task)
}

func updateTask(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])

    var updated Task
    if err := json.NewDecoder(r.Body).Decode(&updated); err != nil {
        http.Error(w, `{"error": "JSON inválido"}`, http.StatusBadRequest)
        return
    }

    mu.Lock()
    defer mu.Unlock()

    for i, task := range tasks {
        if task.ID == id {
            updated.ID = id
            tasks[i] = updated
            json.NewEncoder(w).Encode(updated)
            return
        }
    }
    http.Error(w, `{"error": "tarefa não encontrada"}`, http.StatusNotFound)
}

func deleteTask(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])

    mu.Lock()
    defer mu.Unlock()

    for i, task := range tasks {
        if task.ID == id {
            tasks = append(tasks[:i], tasks[i+1:]...)
            w.WriteHeader(http.StatusNoContent)
            return
        }
    }
    http.Error(w, `{"error": "tarefa não encontrada"}`, http.StatusNotFound)
}
```

## Middleware

Um middleware simples para logging:

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.RequestURI)
        next.ServeHTTP(w, r)
    })
}
```

## Main e rotas

```go
func main() {
    router := mux.NewRouter()

    // Middleware
    router.Use(loggingMiddleware)

    // Rotas
    router.HandleFunc("/tasks", getTasks).Methods("GET")
    router.HandleFunc("/tasks/{id}", getTask).Methods("GET")
    router.HandleFunc("/tasks", createTask).Methods("POST")
    router.HandleFunc("/tasks/{id}", updateTask).Methods("PUT")
    router.HandleFunc("/tasks/{id}", deleteTask).Methods("DELETE")

    log.Println("Servidor rodando na porta 8080...")
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

## Testando a API

Inicie o servidor:

```bash
go run main.go
```

Teste com curl:

```bash
# Criar tarefa
curl -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Aprender Go", "completed": false}'

# Listar tarefas
curl http://localhost:8080/tasks

# Buscar por ID
curl http://localhost:8080/tasks/1

# Atualizar
curl -X PUT http://localhost:8080/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Aprender Go", "completed": true}'

# Deletar
curl -X DELETE http://localhost:8080/tasks/1
```

## O que falta para produção

Essa API é um ponto de partida. Para produção, considere:

- **Persistência**: usar um banco de dados (PostgreSQL, SQLite) em vez de slice em memória
- **Validação**: verificar campos obrigatórios antes de salvar
- **Tratamento de erros**: responses de erro padronizados
- **CORS**: se o frontend rodar em outro domínio
- **Testes**: `testing` package + `httptest` para testar handlers
- **Graceful shutdown**: encerrar conexões pendentes ao parar o servidor

## Conclusão

Go torna a construção de APIs REST um processo simples e direto. A standard library já oferece tudo que você precisa para a maioria dos casos, e o `gorilla/mux` adiciona roteamento flexível quando necessário. A curva de aprendizado é pequena — se você conhece qualquer outra linguagem backend, em poucas horas já consegue construir algo funcional com Go.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
