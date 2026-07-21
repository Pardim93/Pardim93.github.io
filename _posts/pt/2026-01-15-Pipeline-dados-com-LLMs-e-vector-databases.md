---
layout: post
author: Wellington Pardim
title:  "Construindo um pipeline de dados com LLMs e vector databases"
categories: "data-engineering python ai"
position: 43
---

A interseção entre engenharia de dados e IA gerou uma nova arquitetura: **RAG** (Retrieval-Augmented Generation). Em vez de confiar apenas no conhecimento do LLM, você busca informações relevantes em uma base de dados e fornece ao modelo como contexto. Vamos construir isso.

## O que é RAG?

```
Pergunta do usuário → Vector DB (busca) → Contexto relevante → LLM → Resposta
```

1. **Indexar**: converter documentos em vetores (embeddings)
2. **Buscar**: encontrar documentos similares à pergunta
3. **Gerar**: LLM responde usando o contexto encontrado

## Arquitetura completa

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Documents  │     │   Embedding  │     │  Vector DB   │
│   (PDF, TXT, │────→│   Model      │────→│  (ChromaDB)  │
│    MD, HTML) │     │              │     │              │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
┌──────────────┐     ┌──────────────┐            │
│   User       │     │    LLM       │            │
│   Question   │────→│  (OpenAI,    │◄───────────┘
│              │     │   Local)     │  Context
└──────────────┘     └──────────────┘
```

## Instalação

```bash
pip install chromadb openai langchain langchain-community
```

## Indexando documentos

```python
import chromadb
from chromadb.utils import embedding_functions
import os

# Configurar ChromaDB
client = chromadb.PersistentClient(path="./vectordb")

# Usar OpenAI embeddings (ou Sentence Transformers para local)
ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key=os.environ["OPENAI_API_KEY"],
    model_name="text-embedding-3-small",
)

# Criar collection
collection = client.get_or_create_collection(
    name="documentos",
    embedding_function=ef,
)

def indexar_documento(doc_id: str, texto: str, metadata: dict = None):
    """Indexa um documento no vector DB."""
    # Dividir em chunks
    chunks = dividir_em_chunks(texto, tamanho=500, sobreposicao=50)
    
    for i, chunk in enumerate(chunks):
        collection.add(
            ids=[f"{doc_id}_{i}"],
            documents=[chunk],
            metadatas=[metadata or {}],
        )
    
    print(f"Indexado: {doc_id} ({len(chunks)} chunks)")

def dividir_em_chunks(texto: str, tamanho: int = 500, sobreposicao: int = 50) -> list:
    """Divide texto em chunks com sobreposição."""
    chunks = []
    inicio = 0
    while inicio < len(texto):
        fim = inicio + tamanho
        chunk = texto[inicio:fim]
        chunks.append(chunk)
        inicio = fim - sobreposicao
    return chunks
```

## Buscando contexto relevante

```python
def buscar_contexto(pergunta: str, n_results: int = 3) -> str:
    """Busca documentos relevantes para a pergunta."""
    resultados = collection.query(
        query_texts=[pergunta],
        n_results=n_results,
    )
    
    contexto = ""
    for doc, metadata in zip(resultados["documents"][0], resultados["metadatas"][0]):
        fonte = metadata.get("fonte", "documento")
        contexto += f"[Fonte: {fonte}]\n{doc}\n\n"
    
    return contexto
```

## Gerando respostas com RAG

```python
from openai import OpenAI

client_openai = OpenAI()

def responder_com_rag(pergunta: str) -> str:
    """Responde uma pergunta usando RAG."""
    # 1. Buscar contexto
    contexto = buscar_contexto(pergunta)
    
    # 2. Montar prompt
    prompt = f"""Responda à pergunta baseado NO CONTEXTO abaixo.
Se a resposta não estiver no contexto, diga "Não encontrei essa informação nos documentos".

=== CONTEXTO ===
{contexto}
=== FIM DO CONTEXTO ===

PERGUNTA: {pergunta}

RESPOSTA:"""
    
    # 3. Chamar LLM
    resposta = client_openai.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Você é um assistente que responde perguntas baseado em documentos."},
            {"role": "user", "content": prompt},
        ],
        temperature=0.1,  # Baixa temperatura para respostas mais precisas
    )
    
    return resposta.choices[0].message.content
```

## Pipeline completo de indexação

```python
import os
from pathlib import Path

def indexar_diretorio(diretorio: str):
    """Indexa todos os arquivos de um diretório."""
    caminho = Path(diretorio)
    
    for arquivo in caminho.rglob("*"):
        if arquivo.suffix in (".txt", ".md", ".py", ".json"):
            texto = arquivo.read_text(encoding="utf-8")
            if len(texto) > 50:  # Ignorar arquivos muito pequenos
                indexar_documento(
                    doc_id=str(arquivo.relative_to(caminho)),
                    texto=texto,
                    metadata={
                        "fonte": str(arquivo.name),
                        "tipo": arquivo.suffix,
                        "tamanho": len(texto),
                    },
                )

# Indexar documentação do projeto
indexar_diretorio("./docs")
```

## Embeddings locais (sem OpenAI)

```python
from sentence_transformers import SentenceTransformer
import chromadb

# Modelo local (gratuito, roda na sua máquina)
modelo = SentenceTransformer("all-MiniLM-L6-v2")

def embedding_local(texts):
    return modelo.encode(texts).tolist()

# Configurar ChromaDB com embedding local
client = chromadb.PersistentClient(path="./vectordb")
collection = client.get_or_create_collection(
    name="documentos",
    embedding_function=embedding_local,
)
```

## Busca com filtros

```python
def buscar_com_filtro(pergunta: str, tipo_arquivo: str = None):
    """Busca com filtro por tipo de arquivo."""
    where = {}
    if tipo_arquivo:
        where["tipo"] = tipo_arquivo
    
    resultados = collection.query(
        query_texts=[pergunta],
        n_results=5,
        where=where if where else None,
    )
    
    return resultados
```

## Avaliando a qualidade do RAG

```python
def avaliar_rag(perguntas_respostas: list):
    """Avalia a qualidade do sistema RAG."""
    acertos = 0
    total = len(perguntas_respostas)
    
    for pergunta, esperado in perguntas_respostas:
        resposta = responder_com_rag(pergunta)
        
        # Verificar se a resposta contém o esperado
        if esperado.lower() in resposta.lower():
            acertos += 1
        else:
            print(f"ERRO:")
            print(f"  Pergunta: {pergunta}")
            print(f"  Esperado: {esperado}")
            print(f"  Obtido: {resposta[:100]}...")
    
    print(f"\nAcurácia: {acertos}/{total} ({acertos/total*100:.1f}%)")

# Exemplo de avaliação
testes = [
    ("Qual a capital do Brasil?", "Brasília"),
    ("Quem escreveu Dom Casmurro?", "Machado de Assis"),
]
avaliar_rag(testes)
```

## Exemplo real: chatbot de documentação

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/chat", methods=["POST"])
def chat():
    pergunta = request.json["pergunta"]
    
    # Buscar contexto
    contexto = buscar_contexto(pergunta, n_results=3)
    
    # Gerar resposta
    resposta = responder_com_rag(pergunta)
    
    return jsonify({
        "resposta": resposta,
        "fontes": [m["fonte"] for m in collection.query(
            query_texts=[pergunta], n_results=3
        )["metadatas"][0]],
    })

if __name__ == "__main__":
    app.run(debug=True)
```

## Conclusão

RAG é a arquitetura mais prática para aplicações de IA com dados específicos. Em vez de fine-tuning (caro, demorado), você indexa seus documentos em um vector DB e deixa o LLM buscar o contexto relevante.

Comece simples: indexe sua documentação, crie um endpoint de busca, e use o OpenAI para gerar respostas. Evolua para embeddings locais, filtros, e avaliação de qualidade.

O futuro dos dados é IA + dados estruturados. RAG é a ponte entre os dois.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
