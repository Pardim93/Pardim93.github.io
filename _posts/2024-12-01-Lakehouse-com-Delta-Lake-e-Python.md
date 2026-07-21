---
layout: post
author: Wellington Pardim
title:  "Engenharia de dados moderna: Lakehouse com Delta Lake e Python"
categories: "data-engineering python"
position: 39
---

A arquitetura de dados evoluiu: de **Data Warehouse** (SQL, estruturado) para **Data Lake** (qualquer formato, sem controle) e agora para **Lakehouse** — o melhor dos dois mundos. O **Delta Lake** é a tecnologia que torna isso possível.

## O problema

**Data Warehouse** (ex: Redshift, BigQuery):
- Dados estruturados, otimizados para queries
- Caro para armazenar grandes volumes
- Não suporta dados não-estruturados

**Data Lake** (ex: S3, HDFS):
- Barato, suporta qualquer formato
- Sem ACID, sem versionamento
- Difícil de manter qualidade

**Lakehouse** resolve ambos:
- Armazenamento barato (data lake)
- Transações ACID, versionamento, schema enforcement (data warehouse)

## O que é Delta Lake?

Delta Lake é um formato de armazenamento open-source que adiciona uma camada de transações sobre Parquet files em um data lake:

```
Data Lake (S3/ADLS)
└── delta/
    ├── _delta_log/
    │   ├── 00000.json  (transaction log)
    │   ├── 00001.json
    │   └── 00002.json
    ├── part-00000.parquet
    ├── part-00001.parquet
    └── part-00002.parquet
```

O `_delta_log` é o que garante ACID, versionamento e time travel.

## Configuração com PySpark

```bash
pip install pyspark delta-spark
```

```python
from pyspark.sql import SparkSession
from delta import configure_spark_with_delta_pip

builder = (
    SparkSession.builder
    .appName("Lakehouse")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
)

spark = configure_spark_with_delta_pip(builder).getOrCreate()
```

## Escrevendo dados Delta

```python
from pyspark.sql import Row

# Criar DataFrame
dados = [
    Row(id=1, nome="Ana", cidade="SP", valor=150.0),
    Row(id=2, nome="Bob", cidade="RJ", valor=200.0),
    Row(id=3, nome="Carlos", cidade="SP", valor=300.0),
    Row(id=4, nome="Diana", cidade="MG", valor=250.0),
]

df = spark.createDataFrame(dados)

# Escrever como Delta
df.write.format("delta").mode("overwrite").save("/data/vendas")
```

## Time Travel

Cada operação cria uma versão. Você pode consultar dados de qualquer ponto no tempo:

```python
# Versão atual
df_atual = spark.read.format("delta").load("/data/vendas")

# Versão específica
df_v0 = spark.read.format("delta").option("versionAsOf", 0).load("/data/vendas")

# Timestamp específico
df_ontem = (
    spark.read.format("delta")
    .option("timestampAsOf", "2024-01-15")
    .load("/data/vendas")
)

# Ver histórico de versões
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/data/vendas")
delta_table.history().show()
```

## UPSERT (MERGE)

Operação essencial para data warehousing — atualiza existentes e insere novos:

```python
from delta.tables import DeltaTable

# Dados novos
novos_dados = spark.createDataFrame([
    Row(id=2, nome="Bob", cidade="RJ", valor=250.0),  # Atualizar
    Row(id=5, nome="Eve", cidade="BA", valor=180.0),   # Inserir
])

# Delta table existente
delta_table = DeltaTable.forPath(spark, "/data/vendas")

# MERGE (upsert)
(
    delta_table.alias("target")
    .merge(novos_dados.alias("source"), "target.id = source.id")
    .whenMatchedUpdate(set={
        "valor": "source.valor",
        "cidade": "source.cidade",
    })
    .whenNotMatchedInsertAll()
    .execute()
)
```

## Schema Evolution

Adicionar colunas sem reescrever dados:

```python
# DataFrame com nova coluna
df_com_categoria = df.withColumn("categoria", lit("geral"))

# Escrever com merge schema
(
    df_com_categoria.write
    .format("delta")
    .option("mergeSchema", "true")
    .mode("append")
    .save("/data/vendas")
)
```

## Streaming com Delta

Delta Lake suporta streaming nativamente:

```python
# Streaming de dados para Delta
stream = (
    spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "localhost:9092")
    .option("subscribe", "eventos")
    .load()
)

# Processar
from pyspark.sql.functions import from_json, col

schema = "tipo STRING, usuario_id STRING, valor DOUBLE"

processado = (
    stream
    .selectExpr("CAST(value AS STRING)")
    .select(from_json(col("value"), schema).alias("data"))
    .select("data.*")
)

# Escrever em Delta (append)
query = (
    processado.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/checkpoints/vendas")
    .start("/data/vendas_streaming")
)
```

## Qualidade de dados com Delta

```python
# Constraints de qualidade
spark.sql("""
    ALTER TABLE delta.`/data/vendas`
    ADD CONSTRAINT valor_positivo CHECK (valor > 0)
""")

# Tentar inserir dado inválido
df_invalido = spark.createDataFrame([Row(id=6, nome="Teste", cidade="SP", valor=-50)])
df_invalido.write.format("delta").mode("append").save("/data/vendas")
# Erro! Constraint violada
```

## Arquitetura Lakehouse completa

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Sources    │     │   Bronze     │     │   Silver     │
│   (APIs,     │────→│   (Raw)      │────→│   (Cleaned)  │
│    DBs,      │     │   Delta      │     │   Delta      │
│    Files)    │     └──────────────┘     └──────┬───────┘
└──────────────┘                                │
                                                 ▼
                                        ┌──────────────┐
                                        │    Gold       │
                                        │  (Aggregated) │
                                        │   Delta       │
                                        └──────┬───────┘
                                               │
                                        ┌──────┴───────┐
                                        │  Analytics    │
                                        │  (BI, ML)     │
                                        └──────────────┘
```

- **Bronze**: dados brutos, sem transformação
- **Silver**: dados limpos, tipados, deduplicados
- **Gold**: dados agregados, prontos para análise

## Conclusão

Delta Lake é a tecnologia que torna o Lakehouse realidade. Com ACID, time travel, schema evolution e streaming, você tem a confiabilidade de um warehouse com a flexibilidade de um lake.

Comece simples: escreva seus dados em formato Delta em vez de Parquet. Adicione time travel e MERGE conforme necessário. Em pouco tempo, terá uma arquitetura de dados profissional.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
