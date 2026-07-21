---
layout: post
author: Wellington Pardim
title:  "Construindo um CLI com Go e Cobra"
categories: "go"
position: 23
---

Se você já usou `kubectl`, `hugo`, `gh` (GitHub CLI) ou `docker`, usou um tool construído com **Cobra**. É a biblioteca padrão da indústria para criar CLIs em Go. Nesse artigo vamos construir um CLI completo do zero.

## O que é Cobra?

Cobra é uma biblioteca Go para criar aplicações de linha de comando modernas. Ela fornece:

- **Subcomandos** (`app serve`, `app create`, `app delete`)
- **Flags** globais e por comando
- **Help** automático
- **Shell completion** para bash, zsh, fish, powershell

## Instalação

```bash
mkdir meu-cli && cd meu-cli
go mod init meu-cli
go get github.com/spf13/cobra@latest
```

## Estrutura do projeto

```
meu-cli/
├── go.mod
├── main.go
└── cmd/
    ├── root.go
    ├── add.go
    ├── list.go
    └── delete.go
```

## O comando root

Todo CLI Cobra começa com um comando raiz:

```go
// cmd/root.go
package cmd

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
    Use:   "taskman",
    Short: "Gerenciador de tarefas no terminal",
    Long:  "Taskman é uma CLI para gerenciar tarefas do dia a dia direto do terminal.",
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}
```

```go
// main.go
package main

import "meu-cli/cmd"

func main() {
    cmd.Execute()
}
```

## Subcomando: add

```go
// cmd/add.go
package cmd

import (
    "encoding/json"
    "fmt"
    "os"
    "time"

    "github.com/spf13/cobra"
)

type Tarefa struct {
    ID        int       `json:"id"`
    Titulo    string    `json:"titulo"`
    Concluida bool      `json:"concluida"`
    CriadaEm  time.Time `json:"criada_em"`
}

var addCmd = &cobra.Command{
    Use:   "add [título]",
    Short: "Adiciona uma nova tarefa",
    Args:  cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        titulo := args[0]
        prioridade, _ := cmd.Flags().GetString("prioridade")

        tarefas := carregarTarefas()
        nova := Tarefa{
            ID:       len(tarefas) + 1,
            Titulo:   titulo,
            CriadaEm: time.Now(),
        }
        tarefas = append(tarefas, nova)
        salvarTarefas(tarefas)

        fmt.Printf("Tarefa #%d adicionada: %s", nova.ID, titulo)
        if prioridade != "" {
            fmt.Printf(" [prioridade: %s]", prioridade)
        }
        fmt.Println()
    },
}

func init() {
    addCmd.Flags().StringP("prioridade", "p", "", "Prioridade (alta, media, baixa)")
    rootCmd.AddCommand(addCmd)
}
```

## Subcomando: list

```go
// cmd/list.go
package cmd

import (
    "fmt"
    "os"
    "text/tabwriter"

    "github.com/spf13/cobra"
)

var listCmd = &cobra.Command{
    Use:     "list",
    Short:   "Lista todas as tarefas",
    Aliases: []string{"ls"},
    Run: func(cmd *cobra.Command, args []string) {
        tarefas := carregarTarefas()
        if len(tarefas) == 0 {
            fmt.Println("Nenhuma tarefa encontrada.")
            return
        }

        w := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)
        fmt.Fprintln(w, "ID\tTITULO\tSTATUS\tCRIADA EM")
        for _, t := range tarefas {
            status := "pendente"
            if t.Concluida {
                status = "concluída"
            }
            fmt.Fprintf(w, "%d\t%s\t%s\t%s\n",
                t.ID, t.Titulo, status,
                t.CriadaEm.Format("02/01/2006"),
            )
        }
        w.Flush()
    },
}

func init() {
    rootCmd.AddCommand(listCmd)
}
```

## Subcomando: delete

```go
// cmd/delete.go
package cmd

import (
    "fmt"
    "strconv"

    "github.com/spf13/cobra"
)

var deleteCmd = &cobra.Command{
    Use:   "delete [id]",
    Short: "Remove uma tarefa pelo ID",
    Args:  cobra.ExactArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        id, err := strconv.Atoi(args[0])
        if err != nil {
            fmt.Fprintf(os.Stderr, "ID inválido: %s\n", args[0])
            return
        }

        tarefas := carregarTarefas()
        for i, t := range tarefas {
            if t.ID == id {
                tarefas = append(tarefas[:i], tarefas[i+1:]...)
                salvarTarefas(tarefas)
                fmt.Printf("Tarefa #%d removida.\n", id)
                return
            }
        }
        fmt.Fprintf(os.Stderr, "Tarefa #%d não encontrada.\n", id)
    },
}

func init() {
    rootCmd.AddCommand(deleteCmd)
}
```

## Funções auxiliares

```go
// cmd/storage.go
package cmd

import (
    "encoding/json"
    "os"
)

const arquivoTarefas = "tarefas.json"

func carregarTarefas() []Tarefa {
    dados, err := os.ReadFile(arquivoTarefas)
    if err != nil {
        return []Tarefa{}
    }
    var tarefas []Tarefa
    json.Unmarshal(dados, &tarefas)
    return tarefas
}

func salvarTarefas(tarefas []Tarefa) {
    dados, _ := json.MarshalIndent(tarefas, "", "  ")
    os.WriteFile(arquivoTarefas, dados, 0644)
}
```

## Testando

```bash
go build -o taskman .

# Adicionar tarefas
./taskman add "Estudar Go"
./taskman add "Escrever artigo" -p alta

# Listar
./taskman list
# ID  TITULO              STATUS     CRIADA EM
# 1   Estudar Go          pendente   20/12/2020
# 2   Escrever artigo     pendente   20/12/2020

# Deletar
./taskman delete 1

# Ajuda
./taskman --help
./taskman add --help
```

## Shell completion

Cobra gera completions automaticamente:

```bash
# Bash
taskman completion bash > /etc/bash_completion.d/taskman

# Zsh
taskman completion zsh > "${fpath[1]}/_taskman"

# Fish
taskman completion fish > ~/.config/fish/completions/taskman.fish

# PowerShell
taskman completion powershell > taskman.ps1
```

## Global flags

Flags que se aplicam a todos os comandos:

```go
// cmd/root.go
var verbose bool

func init() {
    rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false, "Modo verboso")
}
```

## Conclusão

Cobra é a escolha padrão para CLIs em Go por um motivo: a API é limpa, a documentação é excelente, e o ecossistema é vasto. Ferramentas como kubectl, Hugo, GitHub CLI, e CockroachDB usam Cobra.

Comece com um comando root, adicione subcomandos conforme necessário, e use flags para opções. O Cobra cuida do help, parsing e completions automaticamente.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
