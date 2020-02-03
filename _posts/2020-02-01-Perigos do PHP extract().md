---
layout: post
author: pardim
title:  "Perigos do PHP extract()"
categories: "php security"
position: 2
---
PHP possui várias funções que possibilitam uma maior facilidade de manipulação de dados. Hoje quero falar de uma delas:
a função ```extract()```. A função recebe uma array associativa e transforma todo o par chave e valor da array associativa em variável e valor na [tabela de símbolos](https://pt.wikipedia.org/wiki/Tabela_de_s%C3%ADmbolos). 

Olhando na sua assinatura na documentação para o PHP 7, temos isso aqui: 

```php
extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] ) : int
```
O primeiro parâmetro ``` $array ``` se refere a uma array associativa. O segundo parâmetro ```$flags``` que por default é **EXTR_OVERWRITE** que por definição escreve todos os símbolos mesmo que já existam variáveis com o mesmo nome na tabela, o que é um tanto perigoso, e veremos a razão disso mais a frente. E por final temos o parâmetro  ``` $prefix ``` que por padrão recebe ``` NULL ```, e que pode combater o perigo de sobrescrevimento indesejado. Essa função retorna um inteiro que representa o total de símbolos que foram adicionados na tabela.

A função ```extract()``` esconde uma vulnerabilidade quando usada de forma inadequada com dados vindos de fontes não-confiáveis, e.g: input de usuário. 

Digamos que temos um ``` form.html``` que passa as informações para um ```login.php```: 
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h2>HTML Forms</h2>
    <form action="login.php" method="POST">
      First name:<br>
      <input type="text" id="firstname" name="firstname">
      <br>
      Last name:<br>
      <input type="text" id="lastname" name="lastname">
      <br><br>
      <input type="submit" value="Submit">
    </form> 
</body>
</html>
```
E para lidar com esse ```form.html``` você tenha um ```login.php``` essencialmente assim:
```php
<?php

$credits = 0.0;
extract($_POST);

printf("Welcome to our site: %s %s<br>You have %.2f credits.", $firstname, $lastname, $credits);
```
Que proporciona esse pequeno resultado:   
![Imagem do Login.php](/assets/images/post/{{ page.position }}/extract-login.png)

Essa exibição abre possibilidades para algum usuário mal-intencionado que pode tentar deduzir o nome da variável que carrega o valor dos **credits** dentro da sua aplicação. Provavelmente a sua primeira tentativa será *$credits*. Então ele cria um input no ```form.html``` e seta o seu **id** e **name** como ***credits***. Dessa forma:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h2>HTML Forms</h2>
    <form action="login.php" method="POST">
      First name:<br>
      <input type="text" id="firstname" name="firstname">
      <br>
      Last name:<br>
      <input type="text" id="lastname" name="lastname">
      <br>
      <input type="text" id="credits" name="credits" value="999999999999" hidden>
      <br><br>
      <input type="submit" value="Submit">
    </form> 
</body>
</html>
```
<center><sub > Note o input que fora adicionado. </sub></center>
  
<br>
Ao submetermos esse form temos o seguinte resultado:  
![Imagem do Exploit](/assets/images/post/{{ page.position }}/extract-exploit.png)

Esse é um pequeno exemplo que demonstra a possibilidade de vulnerabilidades que podem ser exploradas com o mal uso desta função. Formas de evitar esse tipo de ocorrência são:
- Não usar essa função quando lidando com fontes não-confiáveis de entrada de dados (*$_POST, $_GET, $_FILES*, etc).  

- Não usar a flag **EXTR_OVERWRITE**, mas usar as outras opções mais seguras que não realizam sobrescrevimento. A flag **EXTR_SKIP** não realiza nenhum sobrescrevimento quando há colisão. 
 
- Usar flags em conjunto com prefixos. Uma combinação interessante é a flag **EXTR_PREFIX_ALL**, com algum prefixo de sua escolha. Essa flag adiciona prefixos em todas as chaves da sua array associativa.  

Caso queira saber mais, consulte a [documentação](https://www.php.net/manual/en/function.extract.php).





