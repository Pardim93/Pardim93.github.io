---
layout: post
author: Wellington Pardim
title:  "Jetpack Compose: UI declarativa para Android"
categories: "mobile android kotlin"
position: 25
---

O **Jetpack Compose** é a nova toolkit do Google para construir UIs nativas no Android. Diferente do sistema tradicional baseado em XML, o Compose usa **funções Kotlin** para descrever a interface. É mais simples, mais expressivo, e eliminou toneladas de boilerplate.

## Por que Compose?

O Android tradicional exigia:

1. XML para layout
2. Java/Kotlin para lógica
3. `findViewById` para conectar os dois
4. Adapters para listas
5. Fragments para telas

O Compose unifica tudo em Kotlin puro:

```kotlin
// Isso é TUDO que você precisa para um botão
Button(onClick = { /* ação */ }) {
    Text("Clique aqui")
}
```

## Componentes básicos (Composable functions)

No Compose, tudo é uma função `@Composable`:

```kotlin
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable

@Composable
fun Saudacao(nome: String) {
    Text(text = "Olá, $nome!")
}
```

## Layouts

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun MeuLayout() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Bem-vindo!",
            style = MaterialTheme.typography.headlineLarge
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = "Escolha uma opção:",
            style = MaterialTheme.typography.bodyLarge
        )
        Spacer(modifier = Modifier.height(24.dp))
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Button(onClick = { /* ação */ }) {
                Text("Opção 1")
            }
            OutlinedButton(onClick = { /* ação */ }) {
                Text("Opção 2")
            }
        }
    }
}
```

## Estado (State)

O Compose re-renderiza automaticamente quando o estado muda:

```kotlin
import androidx.compose.runtime.*

@Composable
fun Contador() {
    var contador by remember { mutableStateOf(0) }

    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        Text(
            text = "Contador: $contador",
            style = MaterialTheme.typography.headlineMedium
        )
        Spacer(modifier = Modifier.height(16.dp))
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(onClick = { contador-- }) {
                Text("-")
            }
            Button(onClick = { contador++ }) {
                Text("+")
            }
        }
    }
}
```

`remember` mantém o estado entre re-renderizações. Sem ele, o contador resetaria toda vez.

## Lista de tarefas completa

```kotlin
data class Tarefa(
    val id: Int,
    val titulo: String,
    val concluida: Boolean = false
)

@Composable
fun ListaTarefas() {
    var tarefas by remember { mutableStateOf(listOf<Tarefa>()) }
    var texto by remember { mutableStateOf("") }

    Column(modifier = Modifier.padding(16.dp)) {
        // Input
        Row(verticalAlignment = Alignment.CenterVertically) {
            OutlinedTextField(
                value = texto,
                onValueChange = { texto = it },
                label = { Text("Nova tarefa") },
                modifier = Modifier.weight(1f)
            )
            Spacer(modifier = Modifier.width(8.dp))
            Button(onClick = {
                if (texto.isNotBlank()) {
                    tarefas = tarefas + Tarefa(
                        id = tarefas.size + 1,
                        titulo = texto
                    )
                    texto = ""
                }
            }) {
                Text("Add")
            }
        }

        Spacer(modifier = Modifier.height(16.dp))

        // Lista
        LazyColumn {
            items(tarefas) { tarefa ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(vertical = 4.dp)
                ) {
                    Row(
                        modifier = Modifier.padding(16.dp),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Checkbox(
                            checked = tarefa.concluida,
                            onCheckedChange = { checked ->
                                tarefas = tarefas.map {
                                    if (it.id == tarefa.id)
                                        it.copy(concluida = checked)
                                    else it
                                }
                            }
                        )
                        Text(
                            text = tarefa.titulo,
                            modifier = Modifier.weight(1f)
                        )
                        IconButton(onClick = {
                            tarefas = tarefas.filter { it.id != tarefa.id }
                        }) {
                            Icon(
                                Icons.Default.Delete,
                                contentDescription = "Remover"
                            )
                        }
                    }
                }
            }
        }
    }
}
```

## Navegação

```kotlin
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController

@Composable
fun MeuApp() {
    val navController = rememberNavController()

    NavHost(navController, startDestination = "home") {
        composable("home") {
            TelaHome(
                onNavigateToDetalhes = { id ->
                    navController.navigate("detalhes/$id")
                }
            )
        }
        composable("detalhes/{id}") { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id")
            TelaDetalhes(id = id)
        }
    }
}
```

## Theme

```kotlin
import androidx.compose.material3.*

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFF58A6FF),
    secondary = Color(0xFF8B949E),
    background = Color(0xFF0D1117),
    surface = Color(0xFF161B22),
)

@Composable
fun MeuTema(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = DarkColorScheme,
        typography = Typography(),
        content = content
    )
}
```

## XML vs Compose

| Aspecto | XML + Views | Compose |
|---------|------------|---------|
| Boilerplate | Alto | Mínimo |
| Estado | Manual (LiveData, StateFlow) | Automático (State) |
| Preview | Precisa rodar o app | Preview no Android Studio |
| Animações | Complexas | Simples e declarativas |
| Learning curve | Baixa (maduro) | Média (conceitos novos) |
| Performance | Excelente | Excelente (equivalente) |

## Conclusão

Jetpack Compose é o futuro do desenvolvimento Android. A Google está investindo pesado — todas as novas APIs são Compose-first, e o XML está em manutenção. Se você está começando em Android agora, comece direto com Compose.

A curva de aprendizado é suave se você já conhece Kotlin. O conceito de "UI como função" é intuitivo depois que você entende o ciclo de recomposição.

Se você vem de React ou Flutter, vai se sentir em casa — o modelo mental é muito similar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
