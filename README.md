# Snake Code

Blog técnico de Wellington Pardim sobre backend, segurança, programação, dados e experimentos de produtividade.

## Sobre

Este é um blog estático construído com [Jekyll](https://jekyllrb.com/) e hospedado no [GitHub Pages](https://pages.github.com/). O design usa um tema escuro customizado inspirado no GitHub.

## Artigos

O blog cobre de 2016 a 2026 com artigos sobre:

- **Python** — versões, features, automação, data engineering
- **Go** — APIs, CLIs, scraping, generics
- **Mobile** — React Native, Flutter, Jetpack Compose, Kotlin Multiplatform
- **Cybersecurity** — ransomware, OWASP, supply chain attacks, autenticação
- **Data Engineering** — Airflow, Kafka, dbt, Delta Lake
- **IA** — MCP, prompt injection, RAG, vector databases

## Desenvolvimento local

```bash
# Instalar dependências
bundle install

# Servir localmente
bundle exec jekyll serve

# Acessar em http://localhost:4000
```

## Estrutura

```
├── _config.yml          # Configuração do Jekyll
├── _layouts/
│   ├── default.html     # Layout base
│   └── post.html        # Layout de artigo
├── _posts/              # Artigos em Markdown
├── assets/css/          # Estilos
├── categories/          # Páginas de categorias
├── index.html           # Página inicial
├── about.html           # Sobre
├── projects.html        # Projetos
└── 404.html             # Página de erro
```

## Licença

Conteúdo &copy; Wellington Pardim. Código sob licença MIT.
