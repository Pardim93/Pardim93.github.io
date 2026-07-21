---
layout: post
author: Wellington Pardim
title:  "Apache Kafka com Python: streaming de dados em tempo real"
categories: "data-engineering python"
position: 35
---

O **Apache Kafka** é a plataforma padrão da indústria para streaming de dados em tempo real. Se você trabalha com dados, é questão de tempo até encontrar Kafka. Nesse artigo vamos entender como funciona e construir producers e consumers em Python.

## O que é Kafka?

Kafka é uma plataforma distribuída para streaming de eventos. Pense nele como um sistema de mensagens com superpoderes:

- **Alto throughput**: milhões de mensagens por segundo
- **Persistência**: mensagens ficam armazenadas por dias/semanas
- **Replay**: consumers podem reler mensagens antigas
- **Distribuído**: tolerante a falhas

## Conceitos fundamentais

```
Producer → Topic (Partition 0, 1, 2) → Consumer Group
```

- **Producer**: envia mensagens para topics
- **Topic**: categoria de mensagens (como uma tabela)
- **Partition**: divisão de um topic (permite paralelismo)
- **Consumer**: lê mensagens de topics
- **Consumer Group**: grupo de consumers que dividem o trabalho
- **Offset**: posição da última mensagem lida por um consumer

## Instalação

```bash
pip install confluent-kafka
```

Para testar localmente, use Docker:

```yaml
# docker-compose.yml
version: "3.8"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.3.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.3.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```bash
docker-compose up -d
```

## Producer

```python
from confluent_kafka import Producer
import json
import time

config = {
    "bootstrap.servers": "localhost:9092",
    "client.id": "meu-producer",
}

producer = Producer(config)

def delivery_callback(err, msg):
    if err:
        print(f"Erro ao enviar: {err}")
    else:
        print(f"Mensagem enviada para {msg.topic()} [{msg.partition()}] offset {msg.offset()}")

# Enviar mensagens
for i in range(10):
    evento = {
        "tipo": "clique",
        "usuario_id": f"user_{i % 3}",
        "pagina": "/produto",
        "timestamp": time.time(),
    }
    
    producer.produce(
        topic="eventos",
        key=f"user_{i % 3}",
        value=json.dumps(evento),
        callback=delivery_callback,
    )
    
    # Flush a cada mensagem (para garantir entrega)
    producer.poll(0)

# Flush final
producer.flush()
print("Todas as mensagens enviadas!")
```

## Consumer

```python
from confluent_kafka import Consumer, KafkaError
import json

config = {
    "bootstrap.servers": "localhost:9092",
    "group.id": "meu-grupo",
    "auto.offset.reset": "earliest",  # Começa do início se não há offset
    "enable.auto.commit": False,
}

consumer = Consumer(config)
consumer.subscribe(["eventos"])

print("Consumindo mensagens... (Ctrl+C para parar)")

try:
    while True:
        msg = consumer.poll(timeout=1.0)
        
        if msg is None:
            continue
        
        if msg.error():
            if msg.error().code() == KafkaError._PARTITION_EOF:
                print(f"Fim da partição {msg.partition()}")
            else:
                print(f"Erro: {msg.error()}")
            continue
        
        # Processar mensagem
        evento = json.loads(msg.value().decode("utf-8"))
        print(f"[{msg.topic()}] {evento}")
        
        # Commit manual após processamento
        consumer.commit(msg)

except KeyboardInterrupt:
    print("Interrompido")
finally:
    consumer.close()
```

## Serialização com Avro

Para schemas estruturados, use Avro:

```python
from confluent_kafka import avro
from confluent_kafka.avro import AvroProducer

# Schema Avro
schema_str = """
{
    "type": "record",
    "name": "Evento",
    "fields": [
        {"name": "tipo", "type": "string"},
        {"name": "usuario_id", "type": "string"},
        {"name": "valor", "type": "double"}
    ]
}
"""

schema = avro.loads(schema_str)

producer = AvroProducer(
    {"bootstrap.servers": "localhost:9092"},
    default_value_schema=schema,
)

evento = {
    "tipo": "compra",
    "usuario_id": "user_123",
    "valor": 99.90,
}

producer.produce(topic="eventos", value=evento)
producer.flush()
```

## Exemplo real: processamento de pagamentos

```python
from confluent_kafka import Consumer, Producer
import json

# Consumer: recebe pedidos
consumer_config = {
    "bootstrap.servers": "localhost:9092",
    "group.id": "pagamentos",
    "auto.offset.reset": "earliest",
}

# Producer: envia resultado
producer_config = {
    "bootstrap.servers": "localhost:9092",
}

consumer = Consumer(consumer_config)
producer = Producer(producer_config)

consumer.subscribe(["pedidos"])

def processar_pagamento(pedido):
    """Simula processamento de pagamento."""
    import random
    if random.random() > 0.1:  # 90% de sucesso
        return {"status": "aprovado", "pedido_id": pedido["id"]}
    return {"status": "recusado", "pedido_id": pedido["id"]}

try:
    while True:
        msg = consumer.poll(timeout=1.0)
        if msg is None:
            continue
        if msg.error():
            continue
        
        pedido = json.loads(msg.value())
        resultado = processar_pagamento(pedido)
        
        # Enviar resultado para outro topic
        producer.produce(
            topic="pagamentos-resultado",
            key=pedido["id"],
            value=json.dumps(resultado),
        )
        producer.flush()
        
        consumer.commit(msg)
        print(f"Pedido {pedido['id']}: {resultado['status']}")

except KeyboardInterrupt:
    pass
finally:
    consumer.close()
```

## Consumer Groups: paralelismo

```python
# Múltiplas instâncias do mesmo consumer group
# Cada instância processa partições diferentes

# Instância 1
Consumer({"group.id": "meu-grupo", ...})
# Processa partição 0 e 1

# Instância 2
Consumer({"group.id": "meu-grupo", ...})
# Processa partição 2 e 3
```

O Kafka distribui partições entre consumers do mesmo grupo automaticamente.

## Dicas práticas

**1. Use keys para ordenação:**
```python
# Mensagens com mesma key vão para mesma partição
producer.produce("topic", key="user_123", value=dados)
# Garante que eventos do user_123 sejam processados em ordem
```

**2. Idempotência:**
```python
# Kafka pode entregar mensagens mais de uma vez
# Seu consumer deve ser idempotente
def processar(evento):
    if ja_processado(evento["id"]):
        return  # Ignorar duplicata
    # ... processar
    marcar_como_processado(evento["id"])
```

**3. Monitoramento:**
```python
# Verifique lag do consumer
from confluent_kafka import Consumer

consumer.list_topics(timeout=10)
```

## Conclusão

Apache Kafka é a espinha dorsal de muitas arquiteturas de dados modernas. Com Python e `confluent-kafka`, você pode rapidamente criar producers e consumers para processar eventos em tempo real.

Comece com um Docker Compose local, crie um producer que envia eventos e um consumer que processa. Depois evolua para schemas Avro, consumer groups e múltiplos tópicos.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
