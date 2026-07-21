---
layout: post
author: Wellington Pardim
title:  "Flutter vs React Native: qual escolher em 2019?"
categories: "mobile flutter react-native"
position: 18
---

Se você quer desenvolver apps mobile com código compartilhado, as duas principais opções são **Flutter** (Google) e **React Native** (Facebook). Ambos são maduros, com comunidades ativas e usados em produção por grandes empresas. Mas qual escolher?

## Visão geral

| Aspecto | Flutter | React Native |
|---------|---------|--------------|
| Linguagem | Dart | JavaScript/TypeScript |
| Criado por | Google | Facebook |
| Primeiro release | 2017 (alpha), 2018 (1.0) | 2015 |
| Rendering | Própria engine (Skia) | Componentes nativos |
| Hot reload | Sim | Sim |
| Base de código | Android + iOS + Web + Desktop | Android + iOS |

## Hello World comparado

### Flutter (Dart)

```dart
import 'package:flutter/material.dart';

void main() => runApp(MeuApp());

class MeuApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Olá Flutter')),
        body: Center(
          child: Text(
            'Olá mundo!',
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```

### React Native (JavaScript)

```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const MeuApp = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.texto}>Olá mundo!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  texto: {
    fontSize: 24,
  },
});

export default MeuApp;
```

## Performance

**Flutter** compila para código nativo (ARM) e usa sua própria engine de rendering (Skia). Isso significa que não há ponte JavaScript — o código roda diretamente no dispositivo. Para animações pesadas e UI complexa, Flutter tende a ser mais consistente.

**React Native** usa uma ponte (bridge) para comunicação entre JavaScript e componentes nativos. Para a maioria dos apps, a performance é excelente. Em casos extremos (animações com 60fps, listas com milhares de itens), pode haver gargalos na ponte.

**Veredito**: Flutter tem vantagem em performance pura. React Native é suficiente para 90% dos casos.

## Ecossistema e bibliotecas

**React Native** tem a vantagem do ecossistema npm — milhões de pacotes JavaScript disponíveis. Se você precisa de uma funcionalidade, provavelmente já existe um package para isso.

**Flutter** tem um ecossistema menor, mas crescente. O [pub.dev](https://pub.dev) é o repositório de pacotes. A qualidade é geralmente alta porque o Google mantém muitos packages oficiais.

## Curva de aprendizado

**React Native** é mais fácil se você já conhece React/JavaScript. A sintaxe é familiar e os conceitos se transferem diretamente.

**Flutter** exige aprender Dart, uma linguagem menos popular. Porém, Dart é fácil de aprender para quem vem de Java, C# ou JavaScript. O conceito de "tudo é widget" é intuitivo depois que você entende.

## Quando escolher Flutter

- **Performance crítica**: animações complexas, jogos, UI customizada
- **Consistência visual**: quer que o app pareça igual em Android e iOS
- **Google ecosystem**: se já usa Firebase, Material Design
- **Desktop e Web**: Flutter suporta Windows, macOS, Linux e Web

## Quando escolher React Native

- **Equipe JavaScript**: se o time já conhece React
- **Ecossistema npm**: precisa de muitas bibliotecas prontas
- **Integração nativa**: precisa de módulos nativos específicos
- **Adoção gradual**: pode ser adicionado a um app nativo existente

## Exemplo real: lista de tarefas

### Flutter

```dart
class TodoList extends StatefulWidget {
  @override
  _TodoListState createState() => _TodoListState();
}

class _TodoListState extends State<TodoList> {
  final List<String> _tarefas = [];
  final _controller = TextEditingController();

  void _adicionar() {
    if (_controller.text.isNotEmpty) {
      setState(() {
        _tarefas.add(_controller.text);
        _controller.clear();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Tarefas')),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: InputDecoration(hintText: 'Nova tarefa'),
                  ),
                ),
                IconButton(icon: Icon(Icons.add), onPressed: _adicionar),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: _tarefas.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(_tarefas[index]),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () => setState(() => _tarefas.removeAt(index)),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### React Native

```javascript
import React, { useState } from 'react';
import { View, Text, TextInput, FlatList, TouchableOpacity, StyleSheet } from 'react-native';

const TodoList = () => {
  const [tarefas, setTarefas] = useState([]);
  const [texto, setTexto] = useState('');

  const adicionar = () => {
    if (texto.trim()) {
      setTarefas([...tarefas, texto]);
      setTexto('');
    }
  };

  const remover = (index) => {
    setTarefas(tarefas.filter((_, i) => i !== index));
  };

  return (
    <View style={styles.container}>
      <View style={styles.inputRow}>
        <TextInput
          style={styles.input}
          value={texto}
          onChangeText={setTexto}
          placeholder="Nova tarefa"
        />
        <TouchableOpacity style={styles.botao} onPress={adicionar}>
          <Text style={styles.botaoTexto}>+</Text>
        </TouchableOpacity>
      </View>
      <FlatList
        data={tarefas}
        keyExtractor={(_, index) => index.toString()}
        renderItem={({ item, index }) => (
          <View style={styles.item}>
            <Text>{item}</Text>
            <TouchableOpacity onPress={() => remover(index)}>
              <Text style={styles.delete}>✕</Text>
            </TouchableOpacity>
          </View>
        )}
      />
    </View>
  );
};
```

## Conclusão

Não existe resposta universal — a escolha depende do contexto:

- **Equipe JavaScript?** → React Native
- **Performance é prioridade?** → Flutter
- **Precisa de Web/Desktop?** → Flutter
- **Precisa de muitas bibliotecas?** → React Native

Ambos são ótimas escolhas e estão em produção em apps que você provavelmente usa todo dia. O melhor é experimentar os dois e ver qual se encaixa melhor no seu fluxo de trabalho.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
