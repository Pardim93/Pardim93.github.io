---
layout: post
author: Wellington Pardim
title:  "Construindo um scraper com Go e Colly"
categories: "go"
position: 15
---

Web scraping é uma habilidade essencial para quem trabalha com dados. Nesse artigo vamos construir um scraper completo em Go usando a biblioteca [Colly](https://github.com/gocolly/colly), que é uma das mais populares da linguagem.

## Por que Go para scraping?

Go se destaca em scraping por dois motivos:

1. **Concorrência nativa** — goroutines permitem raspar múltiplas páginas simultaneamente
2. **Performance** — binários compilados são rápidos, ideal para grandes volumes

## Instalando o Colly

```bash
mkdir scraper-go && cd scraper-go
go mod init scraper-go
go get github.com/gocolly/colly/v2
```

## Scraper básico

Vamos começar com algo simples — extrair títulos e links de uma página:

```go
package main

import (
    "fmt"
    "log"

    "github.com/gocolly/colly/v2"
)

func main() {
    c := colly.NewCollector()

    // Chamado para cada elemento <a> encontrado
    c.OnHTML("a[href]", func(e *colly.HTMLElement) {
        titulo := e.Text
        link := e.Attr("href")
        if titulo != "" && len(titulo) > 10 {
            fmt.Printf("%s\n  %s\n\n", titulo, link)
        }
    })

    // Chamado quando a requisição é feita
    c.OnRequest(func(r *colly.Request) {
        fmt.Println("Visitando:", r.URL.String())
    })

    // Chamado em caso de erro
    c.OnError(func(r *colly.Response, err error) {
        log.Printf("Erro em %s: %v", r.Request.URL, err)
    })

    // Inicia o scraping
    c.Visit("https://news.ycombinator.com")
}
```

Execute:

```bash
go run main.go
```

## Scraper com paginação

Vamos raspar múltiplas páginas, seguindo links de "próxima página":

```go
package main

import (
    "encoding/csv"
    "fmt"
    "log"
    "os"
    "strings"

    "github.com/gocolly/colly/v2"
)

type Artigo struct {
    Titulo string
    Link   string
    Pontos string
}

func main() {
    artigos := []Artigo{}

    c := colly.NewCollector(
        colly.MaxDepth(3), // limite de profundidade
    )

    // Extrair artigos da página
    c.OnHTML(".athing", func(e *colly.HTMLElement) {
        titulo := e.ChildText(".titleline > a")
        link := e.ChildAttr(".titleline > a", "href")
        
        if titulo != "" {
            artigos = append(artigos, Artigo{
                Titulo: titulo,
                Link:   link,
            })
        }
    })

    // Extrair pontos
    c.OnHTML(".score", func(e *colly.HTMLElement) {
        if len(artigos) > 0 {
            artigos[len(artigos)-1].Pontos = e.Text
        }
    })

    // Seguir link de próxima página
    c.OnHTML("a.morelink", func(e *colly.HTMLElement) {
        proxima := e.Attr("href")
        fmt.Printf("Indo para próxima página: %s\n", proxima)
        e.Request.Visit(e.Request.AbsoluteURL(proxima))
    })

    c.OnRequest(func(r *colly.Request) {
        fmt.Println("Visitando:", r.URL.String())
    })

    c.Visit("https://news.ycombinator.com")

    // Salvar em CSV
    salvarCSV(artigos)
    fmt.Printf("\nTotal de artigos: %d\n", len(artigos))
}

func salvarCSV(artigos []Artigo) {
    arquivo, err := os.Create("artigos.csv")
    if err != nil {
        log.Fatal(err)
    }
    defer arquivo.Close()

    escritor := csv.NewWriter(arquivo)
    defer escritor.Flush()

    // Cabeçalho
    escritor.Write([]string{"Titulo", "Link", "Pontos"})

    for _, a := range artigos {
        escritor.Write([]string{a.Titulo, a.Link, a.Pontos})
    }

    fmt.Println("Dados salvos em artigos.csv")
}
```

## Scraper com concorrência

A grande vantagem do Go é a concorrência. Vamos raspar múltiplas URLs simultaneamente:

```go
package main

import (
    "fmt"
    "sync"

    "github.com/gocolly/colly/v2"
)

func rasparURL(url string, wg *sync.WaitGroup, resultados chan<- string) {
    defer wg.Done()

    c := colly.NewCollector()

    c.OnHTML("title", func(e *colly.HTMLElement) {
        resultados <- fmt.Sprintf("%s → %s", url, e.Text)
    })

    c.OnError(func(r *colly.Response, err error) {
        resultados <- fmt.Sprintf("%s → ERRO: %v", url, err)
    })

    c.Visit(url)
}

func main() {
    urls := []string{
        "https://golang.org",
        "https://github.com",
        "https://news.ycombinator.com",
        "https://stackoverflow.com",
        "https://reddit.com",
    }

    resultados := make(chan string, len(urls))
    var wg sync.WaitGroup

    for _, url := range urls {
        wg.Add(1)
        go rasparURL(url, &wg, resultados)
    }

    // Fechar canal quando todas as goroutines terminarem
    go func() {
        wg.Wait()
        close(resultados)
    }()

    for resultado := range resultados {
        fmt.Println(resultado)
    }
}
```

Note o uso de `go` antes de `rasparURL` — isso inicia cada scraping em uma goroutine separada. O `WaitGroup` garante que o programa espere todas terminarem.

## Rate limiting

Para não sobrecarregar o servidor alvo:

```go
c := colly.NewCollector(
    colly.Async(true),
)

// Limita a 2 requisições por segundo
c.Limit(&colly.LimitRule{
    DomainGlob:  "*",
    Parallelism: 2,
    Delay:       500 * time.Millisecond,
})
```

## Tratamento de erros e retry

```go
func visitarComRetry(c *colly.Collector, url string, maxRetries int) {
    for i := 0; i < maxRetries; i++ {
        err := c.Visit(url)
        if err == nil {
            return
        }
        fmt.Printf("Tentativa %d falhou para %s: %v\n", i+1, url, err)
        time.Sleep(time.Duration(i+1) * time.Second)
    }
    fmt.Printf("Falha após %d tentativas: %s\n", maxRetries, url)
}
``## Dicas práticas

**1. User-Agent** — alguns sites bloqueiam requests sem user-agent:

```go
c.OnRequest(func(r *colly.Request) {
    r.Headers.Set("User-Agent", "Mozilla/5.0 (compatible; MeuBot/1.0)")
})
```

**2. Cache** — evite raspar a mesma página duas vezes:

```go
c := colly.NewCollector(
    colly.CacheDir("./cache"),
)
```

**3. Debug** — para entender o que está acontecendo:

```go
c.OnResponse(func(r *colly.Response) {
    fmt.Printf("Status: %d | Tamanho: %d bytes\n", r.StatusCode, len(r.Body))
})
```

**4. Robots.txt** — respeite as regras do site:

```go
c := colly.NewCollector(
    colly.IgnoreRobotsTxt(),
)
// Na verdade, NÃO use IgnoreRobotsTxt em produção
// O Colly respeita robots.txt por padrão, o que é o correto
```

## Conclusão

Go + Colly é uma combinação poderosa para web scraping. A sintaxe limpa do Go, combinada com a concorrência nativa e a API elegante do Colly, permite construir scrapers robustos em poucas linhas.

Para projetos maiores, considere adicionar:

- **Persistência**: salvar dados em PostgreSQL ou MongoDB
- **Proxy rotation**: evitar bloqueio por IP
- **Headless browser**: para sites que dependem de JavaScript (use `chromedp`)
- **Agendamento**: rodar periodicamente com cron ou `time.Ticker`

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
