---
layout: post
author: Wellington Pardim
position: 1
---
O Github adicionou uma nova funcionalidade onde é possível "pinar" seus Gists ao seu perfil. Desta maneira, todo seus projetos que se encontram no Gist podem ser colocados no seu perfil.
Assim é possível, por exemplo, fazer um currículo no formato markdown no Gist e pinar no seu perfil, para que qualquer pessoa que visite seu perfil no Github tenha acesso ao seu currículo.<br>


![Exemplo de um Currículo no Github](/assets/images/post/{{ page.position }}/pinando_gists_image4.gif)
<center><sub> Exemplo de um Gist como gif.</sub></center><br>

Seguindo este exemplo, caso você queira adicionar algum tipo de gif ao seu Github, entre no https://gist.github.com, e crie um novo gist:<br>

![Exemplo de um Currículo no Github](/assets/images/post/{{ page.position }}/pinando_gists_image2.png)
<br>
Crie qualquer tipo de arquivo, pois este será substituido depois.
Com o seu gist criado, o que você precisa fazer agora para adicionar o gif, é fazer um git clone do seu gist. Vá em embed e escolha fazer o clone via SSH ou HTTPS. Feito isso, agora você tendo o repositório local na sua máquina, a única coisa que você precisa fazer é substituir o arquivo que criou por um gif de sua escolha. Faça `git -rm "nome do seu arquivo" `, e adicione o arquivo gif de sua escolha, faça o commit e push.<br>
![Exemplo de um Currículo no Github](/assets/images/post/{{ page.position }}/pinando_gists_image3.png)
<br>
Note que o arquivo antigo foi removido e o gif foi introduzido.
Vá na sua página Gist e perceba que o arquivo foi atualizado com o gif. Para pinar o gif no seu perfil, vá para o seu perfil do Github, e selecione a opção customize your pins, escolha o gist que você acabou de criar e voilá:<br>
![Exemplo de um Currículo no Github](/assets/images/post/{{ page.position }}/
pinando_gists_image1.png)
<br><br>
Seu perfil acaba de ficar mais interessante.
Caso queira, você também pode adicionar um currículo ao seu perfil. Utilize algum template caso queira, e após isso é só você adicionar o arquivo ao seu perfil. Note que para arquivos com markdown, você não irá precisar fazer o clone na sua máquina, pois você poderá editar o arquivo diretamente do Gist, caso queira.
O gist também tem suporte a geojson, você pode por exemplo, usar geojson para mostrar no seu perfil, palestras que você frequentou ou irá frequentar. Você poderá também usar gifs como porta de entrada para um gist seu(que pode ter mais de um arquivo, sendo o primeiro o principal que será mostrado no seu perfil).

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.