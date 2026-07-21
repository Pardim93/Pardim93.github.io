---
layout: post
author: Wellington Pardim
title:  "Construindo um app fullstack com Flutter e Firebase"
categories: "mobile flutter"
position: 38
---

Flutter + Firebase é uma das combinações mais produtivas para construir apps mobile completos. O Flutter cuida da UI e o Firebase cuida do backend: autenticação, banco de dados, storage, e muito mais. Vamos construir um app completo.

## Arquitetura

```
┌─────────────────────────────┐
│          Flutter App         │
│  ┌───────────────────────┐  │
│  │  UI (Widgets)         │  │
│  │  State Management     │  │
│  └───────────┬───────────┘  │
└──────────────┼──────────────┘
               │
┌──────────────┴──────────────┐
│          Firebase            │
│  ┌─────────┐ ┌───────────┐  │
│  │   Auth  │ │ Firestore │  │
│  └─────────┘ └───────────┘  │
│  ┌─────────┐ ┌───────────┐  │
│  │ Storage │ │ Functions │  │
│  └─────────┘ └───────────┘  │
└─────────────────────────────┘
```

## Configuração

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.0
  firebase_auth: ^4.16.0
  cloud_firestore: ^4.14.0
  firebase_storage: ^11.6.0
  provider: ^6.1.1
```

### Inicialização

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MeuApp());
}
```

## Autenticação

```dart
// services/auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class AuthService {
  final FirebaseAuth _auth = FirebaseAuth.instance;

  User? get usuarioAtual => _auth.currentUser;

  Stream<User?> get authStateChanges => _auth.authStateChanges();

  Future<User?> entrarEmailSenha(String email, String senha) async {
    try {
      final resultado = await _auth.signInWithEmailAndPassword(
        email: email,
        password: senha,
      );
      return resultado.user;
    } on FirebaseAuthException catch (e) {
      throw Exception(_traduzirErro(e.code));
    }
  }

  Future<User?> registrarEmailSenha(String email, String senha) async {
    try {
      final resultado = await _auth.createUserWithEmailAndPassword(
        email: email,
        password: senha,
      );
      return resultado.user;
    } on FirebaseAuthException catch (e) {
      throw Exception(_traduzirErro(e.code));
    }
  }

  Future<void> sair() async {
    await _auth.signOut();
  }

  String _traduzirErro(String code) {
    switch (code) {
      case 'user-not-found':
        return 'Usuário não encontrado';
      case 'wrong-password':
        return 'Senha incorreta';
      case 'email-already-in-use':
        return 'E-mail já cadastrado';
      case 'weak-password':
        return 'Senha muito fraca';
      default:
        return 'Erro desconhecido';
    }
  }
}
```

## Firestore: banco de dados

```dart
// services/tarefa_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class TarefaService {
  final CollectionReference _tarefas =
      FirebaseFirestore.instance.collection('tarefas');

  // Criar
  Future<void> criar(String titulo, String usuarioId) async {
    await _tarefas.add({
      'titulo': titulo,
      'concluida': false,
      'usuarioId': usuarioId,
      'criadaEm': FieldValue.serverTimestamp(),
    });
  }

  // Ler (stream em tempo real)
  Stream<QuerySnapshot> listar(String usuarioId) {
    return _tarefas
        .where('usuarioId', isEqualTo: usuarioId)
        .orderBy('criadaEm', descending: true)
        .snapshots();
  }

  // Atualizar
  Future<void> toggleConcluida(String id, bool atual) async {
    await _tarefas.doc(id).update({'concluida': !atual});
  }

  // Deletar
  Future<void> deletar(String id) async {
    await _tarefas.doc(id).delete();
  }
}
```

## State Management com Provider

```dart
// providers/tarefa_provider.dart
import 'package:flutter/foundation.dart';
import '../services/tarefa_service.dart';

class TarefaProvider extends ChangeNotifier {
  final TarefaService _service = TarefaService();
  
  Stream<QuerySnapshot> listarTarefas(String usuarioId) {
    return _service.listar(usuarioId);
  }

  Future<void> adicionar(String titulo, String usuarioId) async {
    await _service.criar(titulo, usuarioId);
  }

  Future<void> toggle(String id, bool atual) async {
    await _service.toggleConcluida(id, atual);
  }

  Future<void> remover(String id) async {
    await _service.deletar(id);
  }
}
```

## Tela de Login

```dart
// screens/login_screen.dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _senhaController = TextEditingController();
  final _auth = AuthService();
  bool _carregando = false;

  Future<void> _entrar() async {
    if (!_formKey.currentState!.validate()) return;
    
    setState(() => _carregando = true);
    try {
      await _auth.entrarEmailSenha(
        _emailController.text,
        _senhaController.text,
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(e.toString())),
      );
    } finally {
      setState(() => _carregando = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Login')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              TextFormField(
                controller: _emailController,
                decoration: InputDecoration(labelText: 'E-mail'),
                validator: (v) => v!.isEmpty ? 'Obrigatório' : null,
              ),
              SizedBox(height: 16),
              TextFormField(
                controller: _senhaController,
                decoration: InputDecoration(labelText: 'Senha'),
                obscureText: true,
                validator: (v) => v!.length < 6 ? 'Mínimo 6 caracteres' : null,
              ),
              SizedBox(height: 24),
              ElevatedButton(
                onPressed: _carregando ? null : _entrar,
                child: _carregando
                    ? CircularProgressIndicator()
                    : Text('Entrar'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Tela principal com lista

```dart
// screens/home_screen.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:provider/provider.dart';
import '../providers/tarefa_provider.dart';
import '../services/auth_service.dart';

class HomeScreen extends StatelessWidget {
  final _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    final provider = Provider.of<TarefaProvider>(context);
    final usuarioId = AuthService().usuarioAtual!.uid;

    return Scaffold(
      appBar: AppBar(
        title: Text('Minhas Tarefas'),
        actions: [
          IconButton(
            icon: Icon(Icons.logout),
            onPressed: () => AuthService().sair(),
          ),
        ],
      ),
      body: StreamBuilder<QuerySnapshot>(
        stream: provider.listarTarefas(usuarioId),
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return Center(child: Text('Erro ao carregar'));
          }
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }

          final tarefas = snapshot.data!.docs;

          if (tarefas.isEmpty) {
            return Center(child: Text('Nenhuma tarefa ainda'));
          }

          return ListView.builder(
            itemCount: tarefas.length,
            itemBuilder: (context, index) {
              final tarefa = tarefas[index];
              final dados = tarefa.data() as Map<String, dynamic>;

              return ListTile(
                leading: Checkbox(
                  value: dados['concluida'],
                  onChanged: (_) => provider.toggle(
                    tarefa.id,
                    dados['concluida'],
                  ),
                ),
                title: Text(
                  dados['titulo'],
                  style: TextStyle(
                    decoration: dados['concluida']
                        ? TextDecoration.lineThrough
                        : null,
                  ),
                ),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () => provider.remover(tarefa.id),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _mostrarDialogo(context, provider, usuarioId),
        child: Icon(Icons.add),
      ),
    );
  }

  void _mostrarDialogo(BuildContext context, TarefaProvider provider, String usuarioId) {
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: Text('Nova Tarefa'),
        content: TextField(
          controller: _controller,
          decoration: InputDecoration(hintText: 'Título'),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: Text('Cancelar'),
          ),
          ElevatedButton(
            onPressed: () {
              if (_controller.text.isNotEmpty) {
                provider.adicionar(_controller.text, usuarioId);
                _controller.clear();
                Navigator.pop(ctx);
              }
            },
            child: Text('Adicionar'),
          ),
        ],
      ),
    );
  }
}
```

## Regras de segurança Firestore

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /tarefas/{tarefaId} {
      allow read, write: if request.auth != null 
        && resource.data.usuarioId == request.auth.uid;
      allow create: if request.auth != null 
        && request.resource.data.usuarioId == request.auth.uid;
    }
  }
}
```

## Conclusão

Flutter + Firebase permite construir apps completos em dias, não semanas. O Firebase cuida de toda a infraestrutura (auth, banco, storage, functions) enquanto você foca na experiência do usuário.

Para apps pequenos e médios, essa combinação é imbatível em produtividade. Para apps maiores, considere adicionar uma camada de abstração sobre o Firebase para facilitar migrações futuras.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
