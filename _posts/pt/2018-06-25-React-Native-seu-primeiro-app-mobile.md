---
layout: post
author: Wellington Pardim
title:  "React Native: seu primeiro app mobile com JavaScript"
categories: "mobile react-native"
position: 13
---

Se você já desenvolve com JavaScript, o React Native permite criar aplicativos mobile nativos para Android e iOS com a mesma base de código. Nesse artigo vamos construir um app simples de clima do zero.

## O que é React Native?

React Native é um framework criado pelo Facebook que permite construir apps mobile usando React. Diferente de apps web que rodam no navegador, React Native renderiza **componentes nativos** — um `<View>` vira uma `UIView` no iOS e um `android.view` no Android.

Vantagens:

- **Código compartilhado** entre Android e iOS (até 90%)
- **Hot reload** — mudanças aparecem instantaneamente
- **Ecossistema rico** — npm packages funcionam no mobile
- **Performance próxima ao nativo** para a maioria dos casos

## Configuração

Instale o CLI do React Native:

```bash
npm install -g react-native-cli
```

Crie um novo projeto:

```bash
npx react-native init MeuClima
cd MeuClima
```

Para rodar:

```bash
# Android (precisa do emulador ou dispositivo conectado)
npx react-native run-android

# iOS (apenas macOS)
npx react-native run-ios
```

## Estrutura básica

O ponto de entrada é `App.js`. Vamos construir um app que mostra a previsão do tempo:

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  FlatList,
  ActivityIndicator,
  TextInput,
  TouchableOpacity,
  SafeAreaView,
  StatusBar,
} from 'react-native';

const API_KEY = 'SUA_CHAVE_API'; // Obtenha em openweathermap.org
const BASE_URL = 'https://api.openweathermap.org/data/2.5';

const App = () => {
  const [cidade, setCidade] = useState('São Paulo');
  const [clima, setClima] = useState(null);
  const [carregando, setCarregando] = useState(false);
  const [erro, setErro] = useState(null);

  const buscarClima = async () => {
    setCarregando(true);
    setErro(null);
    try {
      const resposta = await fetch(
        `${BASE_URL}/weather?q=${cidade}&appid=${API_KEY}&units=metric&lang=pt_br`
      );
      const dados = await resposta.json();

      if (dados.cod !== 200) {
        setErro('Cidade não encontrada');
        setClima(null);
      } else {
        setClima(dados);
      }
    } catch (e) {
      setErro('Erro ao buscar dados');
    } finally {
      setCarregando(false);
    }
  };

  useEffect(() => {
    buscarClima();
  }, []);

  return (
    <SafeAreaView style={estilos.container}>
      <StatusBar barStyle="light-content" />

      <Text style={estilos.titulo}>Clima</Text>

      <View style={estilos.buscaContainer}>
        <TextInput
          style={estilos.input}
          placeholder="Digite a cidade"
          placeholderTextColor="#999"
          value={cidade}
          onChangeText={setCidade}
          onSubmitEditing={buscarClima}
        />
        <TouchableOpacity style={estilos.botao} onPress={buscarClima}>
          <Text style={estilos.botaoTexto}>Buscar</Text>
        </TouchableOpacity>
      </View>

      {carregando && <ActivityIndicator size="large" color="#58a6ff" />}

      {erro && <Text style={estilos.erro}>{erro}</Text>}

      {clima && (
        <View style={estilos.card}>
          <Text style={estilos.cidade}>{clima.name}</Text>
          <Text style={estilos.temperatura}>
            {Math.round(clima.main.temp)}°C
          </Text>
          <Text style={estilos.descricao}>
            {clima.weather[0].description}
          </Text>
          <View style={estilos.detalhes}>
            <Text style={estilos.detalhe}>
              Umidade: {clima.main.humidity}%
            </Text>
            <Text style={estilos.detalhe}>
              Vento: {clima.wind.speed} m/s
            </Text>
          </View>
        </View>
      )}
    </SafeAreaView>
  );
};

const estilos = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0d1117',
    padding: 20,
  },
  titulo: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#ffffff',
    textAlign: 'center',
    marginTop: 20,
    marginBottom: 30,
  },
  buscaContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    backgroundColor: '#161b22',
    borderRadius: 8,
    padding: 12,
    color: '#ffffff',
    fontSize: 16,
    marginRight: 10,
    borderWidth: 1,
    borderColor: '#30363d',
  },
  botao: {
    backgroundColor: '#58a6ff',
    borderRadius: 8,
    paddingHorizontal: 20,
    justifyContent: 'center',
  },
  botaoTexto: {
    color: '#ffffff',
    fontWeight: 'bold',
    fontSize: 16,
  },
  card: {
    backgroundColor: '#161b22',
    borderRadius: 12,
    padding: 24,
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#30363d',
  },
  cidade: {
    fontSize: 24,
    color: '#ffffff',
    fontWeight: '600',
  },
  temperatura: {
    fontSize: 64,
    color: '#58a6ff',
    fontWeight: 'bold',
    marginVertical: 10,
  },
  descricao: {
    fontSize: 18,
    color: '#8b949e',
    textTransform: 'capitalize',
    marginBottom: 20,
  },
  detalhes: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '100%',
  },
  detalhe: {
    color: '#8b949e',
    fontSize: 14,
  },
  erro: {
    color: '#f85149',
    textAlign: 'center',
    fontSize: 16,
    marginTop: 20,
  },
});

export default App;
```

## Conceitos-chave

### useState

Gerencia estado dentro do componente:

```javascript
const [cidade, setCidade] = useState('São Paulo');
// cidade = valor atual
// setCidade = função para atualizar
```

### useEffect

Executa efeitos colaterais (como chamadas de API) após renderização:

```javascript
useEffect(() => {
  buscarClima(); // Executa quando o componente monta
}, []); // Array vazio = executar uma vez
```

### StyleSheet

Similar ao CSS, mas otimizado para mobile:

```javascript
const estilos = StyleSheet.create({
  container: {
    flex: 1,           // preenche todo o espaço disponível
    backgroundColor: '#0d1117',
    padding: 20,
  },
});
```

### Componentes nativos

| React Native | O que vira no nativo |
|-------------|---------------------|
| `<View>` | `UIView` / `android.view.View` |
| `<Text>` | `UILabel` / `android.widget.TextView` |
| `<TextInput>` | `UITextField` / `android.widget.EditText` |
| `<FlatList>` | `UITableView` / `android.widget.RecyclerView` |
| `<TouchableOpacity>` | Botão com feedback de toque |

## Próximos passos

Para evoluir esse app, considere:

- **Navegação**: instale `@react-navigation/native` para múltiplas telas
- **Geolocalização**: use `react-native-geolocation` para clima automático
- **Ícones**: `react-native-vector-icons` para ícones de clima
- **AsyncStorage**: para salvar a cidade preferida localmente
- **Push notifications**: alertas de mudança de clima

## Conclusão

React Native é uma das formas mais rápidas de entrar no desenvolvimento mobile se você já conhece React. O app que construímos acima tem cerca de 150 linhas e já faz uma chamada de API, gerencia estado, trata erros e tem uma interface responsiva.

Para apps mais complexos, a comunidade oferece bibliotecas para tudo: câmera, mapas, pagamentos, notificações. O ecossistema é maduro e battle-tested por apps como Instagram, Facebook e Shopify.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
