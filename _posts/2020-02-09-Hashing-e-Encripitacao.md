---
layout: post
author: Wellington Pardim
title:  "Hashing e Encriptação: suas diferenças"
categories: "security"
position: 3
---
Quando lidando com segurança muito se fala de **Hashing** e **Encriptação**. Algoritmos diferentes que muitas vezes são confundidos entre si, e quando não, as vezes é difícil separar a diferença de cada um.

Queria usar esse artigo para tentar traçar uma diferença entre os dois, e esboçar uma pequena explicação sobre o que é cada um.

**Encriptação** está relacionado ao ato de criptografar uma informação utilizando certos algoritmos(conjuntos de instruções) em conjunto com uma chave privada - também pode ser chamada de senha, se preferir. Essa chave privada é o que o futuramente irá descriptografar a mensagem previamente criptografada.  

Um exemplo histórico é a [Cifra de César](https://pt.wikipedia.org/wiki/Cifra_de_C%C3%A9sar) que era usado em seu tempo para comunicar com os seus generais. A encriptação dava da seguinte maneira. A Cifra de César é um algoritmo simples de criptografia. Para cada letra do alfabeto você adiantava 3 letras.

Digamos que você queira enviar a mensagem "Atacar durante a noite de lua cheia" para os seus generais, para evitar que alguém interceptasse a sua mensagem no meio do caminho e descobrisse o seu comando, você adiantaria toda letra do seu texto para mais 3 casas(nossa chave privada), criptografando a mensagem em "Awdfdu gxudqwh d qrlwh gh oxd fkhld." 

Um general romano tendo a **chave privada** que neste caso é "3 letras", só teria que descriptografar que nesse caso é o processo reverso de adiantar: retroceder. Retroceder 3 letras.

https://www.thesslstore.com/blog/difference-encryption-hashing-salting/

Atualmente, os algoritmos de encriptação modernos incluindo: 

**RSA**: RSA significa Rivest-Shamir-Adlemen, nome de seus criadores, é um algoritmo de criptografia de chave pública (assimétrico) que existe desde 1978 e ainda hoje é amplamente usado. Ele usa a fatoração de números primos para codificar texto simples.

Acompanhe esse vídeo didático sobre o seu funcionamento: 
<a href="https://www.youtube.com/watch?v=4zahvcJ9glg" target="_blank"><img src="https://i.ytimg.com/vi/4zahvcJ9glg/maxresdefault.jpg" 
alt="Video thumbnail" width="360" height="200" border="10" /></a>

Também é importante notar que há 2 métodos de encriptação: 

**Criptografia Assimétrica** - Uma chave criptografa (chave pública) e a outra descriptografa (chave privada). 

**Criptografia Simétrica** -  Cada parte tem sua própria chave que pode criptografar e descriptografar.

---
**Hashing** é a pratica de transformar qualquer entrada em uma saída com um número fixo de caracteres para qualquer input. Em uma função hashing tanto a palavra "abc" quanto a frase "Esse artigo fala sobre as diferenças entre Hashing e Encriptação" possuem saídas com os mesmos números de caracteres. Uma função hashing md5 p.ex. retorna sempre 128 bits, ou 32 caracteres hexadecimais. Enquanto que em encriptação o foco em proteger o transporte de informação, sendo que a parte receptora tanto quanto a que enviou possuem a chave privada para destrancar a informação. Em Hashing não temos a necessidade de reverter a informação original antes da criptografia. Hashing também serve para "carimbar" o estado de um objeto. Por isso é usado o checksum para checar a integridade de um arquivo ou programa. Já que um hashing é feito em seu estado normal, caso você o abaixe e veja que o output do seu checksum está diferente do que a que o site disponibiliza, seu arquivo/programa sofreu modificações - esse processo é conhecido com **code signing**. Além também de seu uso para gravar senhas. 

Em hashing pode ocorrer colisões, ou seja, inputs diferentes podem gerar os mesmos outputs. Isso significa uma vulnerabilidade. Digamos que você possua uma senha 'password' que gere o md5-XXX. Em um hashing com alta colisão, caso uma outra pessoa tente logar na sua conta com um input de senha que também gere o md5-XXX, então ela terá acesso a sua conta.
Por isso já é considerado usar hashings com alta colisão como md5 e SHA-1.

**Salting**
Salting é a técnica de adicionar caractéres em um input, para que ele tenha uma saída diferente do que a original. Isso é util para prevenir ataques de força bruta. Tendo acesso as senhas mais usadas na internet, eu poderia utilizar um dicionário para tentar essas senhas um por uma com um bot, mas com o uso de salting isso inviabiliza esse tipo de ataque. Já que uma senha famosa como 'password', tendo um salt como 'XYZ', ou seja, 'passwordXYX' teria uma saida diferente do que só a hash do 'password'. 

---
Em resumo:

- Criptografia é uma função bidirecional, na qual as informações são embaralhadas de forma que possam ser desembaralhadas posteriormente.

- Hashing é uma função unidirecional em que os dados são mapeados para um valor de comprimento fixo. O hash é usado principalmente para autenticação.

- Sating é uma etapa adicional durante o hash, geralmente vista em associação com senhas com hash, que adiciona um valor adicional ao final da senha que altera o valor do hash produzido.

