---
layout: post
author: Wellington Pardim
title:  "React Native Nova Arquitetura: Fabric, TurboModules e JSI"
categories: "mobile react-native"
position: 33
---

O React Native passou por sua maior mudança arquitetural desde a criação. A **Nova Arquitetura** substitui a ponte (bridge) por um sistema de comunicação direta entre JavaScript e nativo. O resultado: performance muito melhor e APIs mais poderosas.

## O problema da ponte antiga

No React Native clássico, a comunicação JS ↔ Native funcionava assim:

```
JavaScript → Serializa JSON → Ponte → Deserializa → Native
Native → Serializa JSON → Ponte → Deserializa → JavaScript
```

Problemas:
- **Serialização JSON** é lenta para dados grandes
- **Assíncrono** — não há como chamar nativo síncronamente
- **A ponte é um gargalo** — todas as comunicações passam por ela

## A Nova Arquitetura

```
JavaScript ←→ JSI ←→ Native
```

A ponte é substituída por três componentes:

### 1. JSI (JavaScript Interface)

JSI permite que JavaScript acesse objetos nativos **diretamente**, sem serialização:

```cpp
// Antes (Bridge): serializa para JSON
bridge.callNative("MeuModulo", "calcular", JSON.stringify({x: 1, y: 2}))

// JSI: acesso direto ao objeto nativo
global.MeuModulo.calcular({x: 1, y: 2})
```

JSI é uma camada C++ que expõe objetos nativos como JavaScript host objects. Não há serialização.

### 2. Fabric (Novo renderer)

Fabric é o novo sistema de rendering que substitui o antigo UIManager:

```javascript
// Fabric: rendering síncrono quando necessário
function MeuComponente() {
  const [tamanho, setTamanho] = useState(0);
  
  // Fabric pode medir layout síncronamente
  const medir = () => {
    ref.current.measure((x, y, width, height) => {
      setTamanho(width);
    });
  };
  
  return <View ref={ref} onLayout={medir} />;
}
```

Vantagens do Fabric:
- **Rendering síncrono** para operações críticas
- **Melhor suporte a concurrent features** do React 18
- **Componentes customizados** com C++ nativo

### 3. TurboModules

TurboModules substituem os antigos NativeModules com carregamento lazy e tipagem forte:

```javascript
// Antigo NativeModule: carregava tudo na inicialização
import { NativeModules } from 'react-native';
const { MeuModulo } = NativeModules;

// TurboModule: carrega sob demanda
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

interface Spec extends TurboModule {
  calcular(x: number, y: number): number;
  processar(dados: string): Promise<string>;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MeuModulo');
```

## Codegen

A Codegen gera o código de ligação (binding) automaticamente a partir das specs:

```javascript
// NativeMeuModulo.js (spec)
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  +hashSenha(senha: string, salt: string): string;
  +verificarSenha(senha: string, hash: string): boolean;
}

export default (TurboModuleRegistry.getEnforcing<Spec>('MeuModulo'): Spec);
```

A Codegen gera:
- Objeto C++ para iOS
- Interface Java para Android
- Bindings JavaScript

## Migrando para a Nova Arquitetura

### 1. Atualize o React Native

```bash
npx react-native upgrade
```

### 2. Habilite a Nova Arquitetura

```javascript
// android/gradle.properties
newArchEnabled=true

// ios/Podfile
:fabric_enabled => true
```

### 3. Migre NativeModules

```javascript
// Antes
import { NativeModules } from 'react-native';
const { Crypto } = NativeModules;
export default Crypto;

// Depois
import { TurboModuleRegistry } from 'react-native';
import type { TurboModule } from 'react-native';

interface Spec extends TurboModule {
  hashSenha(senha: string, salt: string): string;
}

export default TurboModuleRegistry.getEnforcing<Spec>('Crypto');
```

## Performance comparada

| Operação | Bridge antiga | Nova Arquitetura |
|----------|--------------|-----------------|
| Chamada de função nativa | ~5ms | ~0.1ms |
| Transferência de 1MB | ~50ms | ~1ms |
| Layout measurement | Assíncrono | Síncrono disponível |
| Carregamento de módulos | Eager (tudo) | Lazy (sob demanda) |

## Quando migrar

- **Novos projetos**: comece com a Nova Arquitetura
- **Projetos existentes**: migre gradualmente — a compatibilidade com a antiga é mantida
- **Bibliotecas**: verifique se suas dependências suportam a Nova Arquitetura

## Conclusão

A Nova Arquitetura do React Native é uma reescrita fundamental que resolve os problemas de performance da ponte. O JSI permite comunicação direta, o Fabric melhora o rendering, e os TurboModules trazem carregamento lazy e tipagem forte.

Se você usa React Native, a migração é inevitável — o time do React Native está investindo tudo na Nova Arquitetura. Comece a experimentar em projetos novos e migre existentes gradualmente.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
