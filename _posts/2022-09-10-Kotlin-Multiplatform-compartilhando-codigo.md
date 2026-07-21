---
layout: post
author: Wellington Pardim
title:  "Kotlin Multiplatform: compartilhando código entre Android e iOS"
categories: "mobile kotlin"
position: 30
---

O sonho de escrever código uma vez e rodar em múltiplas plataformas existe há décadas. O **Kotlin Multiplatform** (KMP) é a abordagem mais pragmática que vi até agora — em vez de substituir a UI nativa, ele compartilha **apenas a lógica de negócio**.

## O que é Kotlin Multiplatform?

KMP permite compartilhar código Kotlin entre plataformas:

- **Android** (nativo)
- **iOS** (nativo, via Kotlin/Native)
- **Desktop** (JVM)
- **Web** (Kotlin/JS ou Kotlin/Wasm)

O diferencial do KMP é que ele **não substitui a UI nativa**. Você compartilha a lógica (repositories, use cases, models, networking) e mantém a UI nativa (Jetpack Compose no Android, SwiftUI no iOS).

## Arquitetura

```
┌─────────────────────────────────────┐
│         Shared Module (KMP)         │
│  ┌───────────────────────────────┐  │
│  │  Models / Data Classes        │  │
│  │  API Client (Ktor)            │  │
│  │  Database (SQLDelight)        │  │
│  │  Use Cases / Business Logic   │  │
│  │  Repository Pattern           │  │
│  └───────────────────────────────┘  │
└──────────┬──────────────┬───────────┘
           │              │
    ┌──────┴──────┐ ┌─────┴──────┐
    │   Android   │ │    iOS     │
    │  Jetpack    │ │  SwiftUI   │
    │  Compose    │ │            │
    └─────────────┘ └────────────┘
```

## Configuração do projeto

### build.gradle.kts (shared module)

```kotlin
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization") version "1.7.20"
    id("com.android.library")
}

kotlin {
    android()
    
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach {
        it.binaries.framework {
            baseName = "shared"
        }
    }

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("io.ktor:ktor-client-core:2.2.0")
                implementation("io.ktor:ktor-client-content-negotiation:2.2.0")
                implementation("io.ktor:ktor-serialization-kotlinx-json:2.2.0")
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4")
                implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.4.1")
                implementation("com.squareup.sqldelight:runtime:1.5.5")
            }
        }
        
        val androidMain by getting {
            dependencies {
                implementation("io.ktor:ktor-client-okhttp:2.2.0")
                implementation("com.squareup.sqldelight:android-driver:1.5.5")
            }
        }
        
        val iosMain by getting {
            dependencies {
                implementation("io.ktor:ktor-client-darwin:2.2.0")
                implementation("com.squareup.sqldelight:native-driver:1.5.5")
            }
        }
    }
}
```

## Código compartilhado

### Model

```kotlin
// shared/src/commonMain/kotlin/Models.kt
import kotlinx.serialization.Serializable

@Serializable
data class Tarefa(
    val id: Int,
    val titulo: String,
    val concluida: Boolean = false
)
```

### API Client

```kotlin
// shared/src/commonMain/kotlin/TarefaApi.kt
import io.ktor.client.*
import io.ktor.client.call.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.request.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

class TarefaApi {
    private val client = HttpClient {
        install(ContentNegotiation) {
            json(Json { ignoreUnknownKeys = true })
        }
    }

    suspend fun buscarTarefas(): List<Tarefa> {
        return client.get("https://api.exemplo.com/tarefas").body()
    }

    suspend fun criarTarefa(titulo: String): Tarefa {
        return client.post("https://api.exemplo.com/tarefas") {
            setBody(mapOf("titulo" to titulo))
        }.body()
    }
}
```

### Repository

```kotlin
// shared/src/commonMain/kotlin/TarefaRepository.kt
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow

class TarefaRepository(private val api: TarefaApi) {
    private val _tarefas = MutableStateFlow<List<Tarefa>>(emptyList())
    val tarefas: Flow<List<Tarefa>> = _tarefas

    suspend fun carregar() {
        _tarefas.value = api.buscarTarefas()
    }

    suspend fun adicionar(titulo: String) {
        val nova = api.criarTarefa(titulo)
        _tarefas.value = _tarefas.value + nova
    }
}
```

## Consumindo no Android

```kotlin
// Android - ViewModel
class TarefaViewModel : ViewModel() {
    private val api = TarefaApi()
    private val repo = TarefaRepository(api)
    
    val tarefas = repo.tarefas.stateIn(
        viewModelScope, SharingStarted.Lazily, emptyList()
    )

    fun carregar() {
        viewModelScope.launch { repo.carregar() }
    }

    fun adicionar(titulo: String) {
        viewModelScope.launch { repo.adicionar(titulo) }
    }
}

// Android - Composable
@Composable
fun TarefaScreen(viewModel: TarefaViewModel = viewModel()) {
    val tarefas by viewModel.tarefas.collectAsState()
    
    LazyColumn {
        items(tarefas) { tarefa ->
            Text(tarefa.titulo)
        }
    }
}
```

## Consumindo no iOS (Swift)

```swift
// iOS - ViewModel
import shared

class TarefaViewModel: ObservableObject {
    private let api = TarefaApi()
    private let repo: TarefaRepository
    
    @Published var tarefas: [Tarefa] = []
    
    init() {
        repo = TarefaRepository(api: api)
        
        // Coletar Flow do Kotlin
        repo.tarefas.collect { tarefas in
            DispatchQueue.main.async {
                self.tarefas = tarefas
            }
        }
    }
    
    func carregar() {
        Task { await repo.carregar() }
    }
}

// iOS - SwiftUI
struct TarefaView: View {
    @StateObject private var viewModel = TarefaViewModel()
    
    var body: some View {
        List(viewModel.tarefas, id: \.id) { tarefa in
            Text(tarefa.titulo)
        }
        .onAppear { viewModel.carregar() }
    }
}
```

## Vantagens sobre React Native / Flutter

| Aspecto | KMP | React Native | Flutter |
|---------|-----|-------------|---------|
| UI nativa | Sim | Sim | Não |
| Lógica compartilhada | Sim | Parcial | Sim |
| Performance | Nativa | Ponte JS | Nativa |
| Acesso nativo | Direto | Via bridge | Via plugins |
| Learning curve | Kotlin | JavaScript | Dart |

## Quando usar KMP

- **Equipe que já conhece Kotlin/Android**
- **Apps que precisam de UI nativa** em cada plataforma
- **Lógica de negócio complexa** que vale a pena compartilhar
- **Apps com muitas integrações nativas** (câmera, Bluetooth, pagamentos)

## Conclusão

Kotlin Multiplatform é a abordagem mais pragmática para código compartilhado. Em vez de tentar substituir a UI nativa (como Flutter) ou usar uma ponte JavaScript (como React Native), ele compartilha apenas o que faz sentido: a lógica de negócio.

Se você já desenvolve Android com Kotlin, o KMP é natural — você continua escrevendo Kotlin, mas agora o mesmo código roda no iOS. A JetBrains está investindo pesado e o ecossistema está amadurecendo rápido.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
