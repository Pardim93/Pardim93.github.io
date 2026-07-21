---
layout: post
author: Wellington Pardim
title:  "MCP: integrando IA com ferramentas externas"
categories: "python ai"
position: 40
---

O **Model Context Protocol (MCP)** é um protocolo aberto que permite a LLMs (Large Language Models) interagirem com ferramentas externas de forma padronizada. Pense como "USB para IA" — qualquer ferramenta que implemente o protocolo funciona com qualquer LLM que o suporte.

## O que é MCP?

MCP define um padrão para comunicação entre:

- **MCP Client**: o LLM ou aplicação que quer usar ferramentas
- **MCP Server**: o serviço que expõe ferramentas e recursos

```
┌─────────────┐     MCP      ┌─────────────┐
│  LLM/Agent  │◄────────────►│  MCP Server  │
│  (Client)   │              │  (Tools,     │
│             │              │   Resources) │
└─────────────┘              └─────────────┘
```

## Por que MCP?

Sem MCP, cada integração IA é custom:

```
LLM + Slack = código custom
LLM + GitHub = código custom  
LLM + Database = código custom
```

Com MCP:

```
LLM + Slack MCP Server = funciona
LLM + GitHub MCP Server = funciona
LLM + Database MCP Server = funciona
```

## Implementando um MCP Server em Python

### Instalação

```bash
pip install mcp
```

### Server básico

```python
# mcp_server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import json

server = Server("meu-mcp-server")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="calcular",
            description="Realiza operações matemáticas básicas",
            inputSchema={
                "type": "object",
                "properties": {
                    "expressao": {
                        "type": "string",
                        "description": "Expressão matemática (ex: '2 + 2')",
                    }
                },
                "required": ["expressao"],
            },
        ),
        Tool(
            name="buscar_clima",
            description="Busca a previsão do tempo para uma cidade",
            inputSchema={
                "type": "object",
                "properties": {
                    "cidade": {
                        "type": "string",
                        "description": "Nome da cidade",
                    }
                },
                "required": ["cidade"],
            },
        ),
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "calcular":
        try:
            resultado = eval(arguments["expressao"])  # Apenas para demo!
            return [TextContent(type="text", text=f"Resultado: {resultado}")]
        except Exception as e:
            return [TextContent(type="text", text=f"Erro: {e}")]
    
    elif name == "buscar_clima":
        cidade = arguments["cidade"]
        # Simulação — em produção, chamaria uma API real
        clima = {"São Paulo": "25°C, ensolarado", "Rio": "30°C, nublado"}
        resultado = clima.get(cidade, "Cidade não encontrada")
        return [TextContent(type="text", text=f"Clima em {cidade}: {resultado}")]
    
    return [TextContent(type="text", text=f"Ferramenta desconhecida: {name}")]

async def main():
    async with stdio_server() as (read, write):
        await server.run(read, write, server.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## MCP Server com recursos

Além de tools, MCP suporta **resources** (dados que o LLM pode ler):

```python
@server.list_resources()
async def list_resources():
    return [
        Resource(
            uri="file:///config.json",
            name="Configuração do sistema",
            description="Arquivo de configuração atual",
            mimeType="application/json",
        ),
    ]

@server.read_resource()
async def read_resource(uri: str):
    if uri == "file:///config.json":
        config = {"ambiente": "producao", "versao": "1.0.0"}
        return json.dumps(config, indent=2)
    raise ValueError(f"Recurso não encontrado: {uri}")
```

## MCP Server com prompts

```python
@server.list_prompts()
async def list_prompts():
    return [
        Prompt(
            name="analise_codigo",
            description="Analisa um trecho de código em busca de problemas",
            arguments=[
                PromptArgument(
                    name="codigo",
                    description="O código para analisar",
                    required=True,
                )
            ],
        ),
    ]

@server.get_prompt()
async def get_prompt(name: str, arguments: dict):
    if name == "analise_codigo":
        codigo = arguments["codigo"]
        return GetPromptResult(
            messages=[
                PromptMessage(
                    role="user",
                    content=TextContent(
                        type="text",
                        text=f"Analise este código e sugira melhorias:\n\n```python\n{codigo}\n```",
                    ),
                )
            ]
        )
```

## Usando com Claude

O Anthropic Claude suporta MCP nativamente. Configure no `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "meu-server": {
      "command": "python",
      "args": ["/path/to/mcp_server.py"]
    }
  }
}
```

## Exemplo real: MCP Server para banco de dados

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import sqlite3

server = Server("db-mcp")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="query_sql",
            description="Executa uma query SQL SELECT no banco de dados",
            inputSchema={
                "type": "object",
                "properties": {
                    "sql": {
                        "type": "string",
                        "description": "Query SQL SELECT",
                    }
                },
                "required": ["sql"],
            },
        ),
        Tool(
            name="listar_tabelas",
            description="Lista todas as tabelas do banco",
            inputSchema={"type": "object", "properties": {}},
        ),
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    conn = sqlite3.connect("meubanco.db")
    
    if name == "query_sql":
        sql = arguments["sql"]
        if not sql.strip().upper().startswith("SELECT"):
            return [TextContent(type="text", text="Erro: apenas SELECT permitido")]
        
        cursor = conn.execute(sql)
        colunas = [desc[0] for desc in cursor.description]
        linhas = cursor.fetchall()
        
        resultado = f"Colunas: {colunas}\n"
        for linha in linhas[:10]:  # Limitar a 10 linhas
            resultado += f"{linha}\n"
        
        return [TextContent(type="text", text=resultado)]
    
    elif name == "listar_tabelas":
        cursor = conn.execute(
            "SELECT name FROM sqlite_master WHERE type='table'"
        )
        tabelas = [row[0] for row in cursor.fetchall()]
        return [TextContent(type="text", text=f"Tabelas: {tabelas}")]
    
    conn.close()
```

## Ecossistema MCP

Servers MCP já disponíveis:

| Server | O que faz |
|--------|----------|
| GitHub | Repositórios, issues, PRs |
| Slack | Mensagens, canais |
| PostgreSQL | Queries em banco |
| Filesystem | Ler/esverver arquivos |
| Puppeteer | Automação de navegador |
| Google Drive | Arquivos e pastas |
| Notion | Páginas e databases |

## Conclusão

MCP é o protocolo que vai padronizar como IAs interagem com o mundo. Em vez de integrações custom para cada serviço, temos um padrão aberto que qualquer LLM e qualquer ferramenta pode implementar.

Comece criando um MCP Server simples para uma ferramenta que você usa. Em poucas horas, qualquer LLM que suporte MCP poderá usar sua ferramenta.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
