---
layout: post
author: Wellington Pardim
title:  "Construindo um bot de automação com Python e Playwright"
categories: "python automation"
position: 34
---

Automação de navegadores é uma habilidade poderosa. O **Playwright** é a ferramenta mais moderna para isso — mais rápida e confiável que o Selenium, com suporte a Chromium, Firefox e WebKit. Vamos construir um bot completo.

## Por que Playwright?

- **Mais rápido** que Selenium (comunicação direta, sem WebDriver)
- **Auto-wait** — espera elementos aparecerem automaticamente
- **Multi-browser** — Chromium, Firefox, WebKit
- **Async e sync** — suporta ambos os modos
- **Screenshots e vídeos** — gravar sessões facilmente
- **Codegen** — grave interações e gere código automaticamente

## Instalação

```bash
pip install playwright
playwright install
```

## Primeiro script: login automatizado

```python
from playwright.sync_api import sync_playwright

def fazer_login():
    with sync_playwright() as p:
        # Iniciar navegador (headless=False para ver)
        navegador = p.chromium.launch(headless=False)
        pagina = navegador.new_page()

        # Navegar até a página de login
        pagina.goto("https://exemplo.com/login")

        # Preencher formulário
        pagina.fill('input[name="email"]', "meu@email.com")
        pagina.fill('input[name="senha"]', "minhaSenha123")
        pagina.click('button[type="submit"]')

        # Esperar navegação completar
        pagina.wait_for_url("**/dashboard")

        print(f"Login realizado! URL: {pagina.url}")

        # Screenshot como prova
        pagina.screenshot(path="dashboard.png")

        navegador.close()

fazer_login()
```

## Auto-wait: a mágica do Playwright

Diferente do Selenium, você não precisa de `time.sleep()` ou `WebDriverWait`:

```python
# Playwright espera automaticamente o elemento aparecer
pagina.click("#botao-dinamico")  # Espera até estar visível e clicável

# Se precisar de espera explícita
pagina.wait_for_selector(".resultado", timeout=10000)

# Esperar por texto
pagina.wait_for_function("document.querySelector('#status').textContent === 'Completo'")
```

## Scraping com Playwright

Para sites que dependem de JavaScript:

```python
from playwright.sync_api import sync_playwright
import json

def raspar_site():
    with sync_playwright() as p:
        navegador = p.chromium.launch(headless=True)
        pagina = navegador.new_page()

        pagina.goto("https://news.ycombinator.com")

        # Extrair dados usando evaluate
        artigos = pagina.evaluate("""
            () => {
                const rows = document.querySelectorAll('.athing');
                return Array.from(rows).map(row => ({
                    titulo: row.querySelector('.titleline > a')?.textContent,
                    link: row.querySelector('.titleline > a')?.href,
                })).filter(a => a.titulo);
            }
        """)

        print(f"Extraídos {len(artigos)} artigos")
        
        with open("artigos.json", "w") as f:
            json.dump(artigos, f, indent=2, ensure_ascii=False)

        navegador.close()

raspar_site()
```

## Bot de monitoramento de preços

```python
from playwright.sync_api import sync_playwright
import time
import json
from datetime import datetime

def monitorar_preco(url, seletor_preco, preco_alvo):
    """Monitora o preço de um produto e alerta quando cai abaixo do alvo."""
    
    historico = []

    with sync_playwright() as p:
        navegador = p.chromium.launch(headless=True)
        pagina = navegador.new_page()
        
        while True:
            try:
                pagina.goto(url, timeout=30000)
                
                # Extrair preço
                texto_preco = pagina.text_content(seletor_preco)
                preco = float(
                    texto_preco
                    .replace("R$", "")
                    .replace(".", "")
                    .replace(",", ".")
                    .strip()
                )
                
                registro = {
                    "data": datetime.now().isoformat(),
                    "preco": preco,
                }
                historico.append(registro)
                
                print(f"[{registro['data']}] R$ {preco:.2f}")
                
                if preco <= preco_alvo:
                    print(f"ALERTA: Preço caiu para R$ {preco:.2f}!")
                    # Aqui você poderia enviar e-mail, notificação, etc.
                    break
                
                # Salvar histórico
                with open("historico_precos.json", "w") as f:
                    json.dump(historico, f, indent=2)
                
                # Esperar 5 minutos
                time.sleep(300)
                
            except Exception as e:
                print(f"Erro: {e}")
                time.sleep(60)

monitorar_preco(
    url="https://loja.com/produto",
    seletor_preco=".preco-atual",
    preco_alvo=99.99
)
```

## Formulários complexos

```python
def preencher_formulario():
    with sync_playwright() as p:
        navegador = p.chromium.launch(headless=False)
        pagina = navegador.new_page()
        pagina.goto("https://exemplo.com/cadastro")

        # Texto
        pagina.fill("#nome", "Wellington")
        pagina.fill("#email", "well@email.com")

        # Select
        pagina.select_option("#estado", "SP")

        # Checkbox
        pagina.check("#aceito_termos")

        # Radio
        pagina.click('input[name="plano"][value="premium"]')

        # Upload de arquivo
        pagina.set_input_files("#documento", "documento.pdf")

        # Data
        pagina.fill("#data_nascimento", "1993-01-15")

        # Submeter
        pagina.click("#submit")
        pagina.wait_for_url("**/sucesso")

        navegador.close()
```

## Codegen: grave e gere código

```bash
# Abre o navegador e grava suas ações
playwright codegen https://exemplo.com
```

Isso abre um navegador e gera código Python conforme você interage. Útil para descobrir seletores CSS.

## Screenshots e vídeos

```python
def gravar_sessao():
    with sync_playwright() as p:
        navegador = p.chromium.launch(headless=False)
        
        # Contexto com gravação de vídeo
        contexto = navegador.new_context(
            record_video_dir="./videos",
            viewport={"width": 1280, "height": 720}
        )
        pagina = contexto.new_page()
        
        pagina.goto("https://exemplo.com")
        pagina.screenshot(path="screenshot1.png", full_page=True)
        
        # ... interações ...
        
        pagina.screenshot(path="screenshot2.png")
        contexto.close()
        navegador.close()
```

## Async Playwright

Para aplicações async (FastAPI, aiohttp):

```python
import asyncio
from playwright.async_api import async_playwright

async def raspar_async(urls):
    async with async_playwright() as p:
        navegador = await p.chromium.launch(headless=True)
        
        tarefas = []
        for url in urls:
            pagina = await navegador.new_page()
            tarefas.append(processar_pagina(pagina, url))
        
        resultados = await asyncio.gather(*tarefas)
        await navegador.close()
        return resultados

async def processar_pagina(pagina, url):
    await pagina.goto(url)
    titulo = await pagina.title()
    return {"url": url, "titulo": titulo}

urls = ["https://site1.com", "https://site2.com", "https://site3.com"]
resultados = asyncio.run(raspar_async(urls))
```

## Conclusão

Playwright é a ferramenta definitiva para automação de navegadores em Python. É mais rápido, mais confiável e com melhor API que o Selenium. Use para:

- **Scraping** de sites com JavaScript
- **Automação** de formulários e workflows
- **Testes E2E** de aplicações web
- **Monitoramento** de preços, disponibilidade
- **Bots** de interação com websites

Comece com o `playwright codegen` para explorar sites e gerar código. Depois refine para scripts robustos com tratamento de erros.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
