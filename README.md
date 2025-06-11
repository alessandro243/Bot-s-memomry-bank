<h1 align='center' style ="font-size: 18px"><b>Extesão do projeto república: Implementação de memória em banco de dados</b></h1>

Como bem vimos, os cômodos de nossa república comunicam-se através de arquivos de texto, isso funciona bem com as consultas, que na maior parte das vezes são rápidas e simples, usando apenas um dado simples, como dia, segundos ou um dado booleano. Só que isso não serve para estruturas de dados mais complexos, extensos e não uniformes, como listas de dicionarios grandes, o que da forma que fazemos atualmente, é um problema quando consideramos desenvolver uma memória consistente para um NPC, pois não é escalável e não oferece segurança nos dados.

No modo que está, você só passa a ter algum tipo de registro em memória a partir do momento em que fala com o Cordano no Bar, ele então adiciona você a um registro de usuários em um arquivo .json. Nesse arquivo terá informações como nome do usuário, se você está interagindo pela primeira vez e sua lista de itens. A ideia era que cada bot usaria esse .json, onde teria, para cada item do usuário, uma lista de bots que interagem com esse item e após a interação ele apaga sem nome dessa lista inutilizando determinada linha de diálogo.
Mas isso passa a não funcionar tão bem quando consideramos um numero grande de usuários e vários itens, pois o arquivo ficaria cada vez mais pesado, confuso e não seria uniforme. Ainda devemos considerar que determinado bot pode acabar alterando uma informação que outro bot usaria causando erros de narrativa.

Dessa forma nós planejamos implementar uma rede de tabelas para cada bot, estabelecendo uma memória robusta e persistente e com suporte a várias conexões para cada personagem. Então ao entrar no servidor pela primeira vez o usuário será recepcionado por um bot administrativo, que o fará perguntas para otmizar a experiência do usuário, perguntas como nome, estilo de música preferido. Ele então registrará as informações na tabela de usuários com um inventário para itens, dessa forma os bots poderão acessar esse inventario para gerenciar suas próprias frases e ações.

<hr>

### ⚙️ Tecnologias que serão utilizadas 

Linguagens, ferramentas e bibliotecas utilizadas no desenvolvimento do projeto:

* [Python:](https://www.python.org/) A linguagem base para integrar as tabelas ao projeto.
* [MySQL:](https://www.mysql.com/) Será o SGBD escolhido, por ter APIs com documentação amplamente desenvolvida para integração com python e ter suporte a multiplas conexões.
* [pymysql:](https://pymysql.readthedocs.io/en/latest/) API usada para integrar MySQL ao projeto.
* [sqlalchemy:](https://www.sqlalchemy.org/) API usada para integrar MySQL ao projeto.
* [DBeaver:](https://dbeaver.io/) Programa usado para melhor visualização dos dados nas tabelas.

<hr>

<details>
  <summary>Planejamento da memória do Mingau</summary>
  <br>

> O desenvolvimento de uma memória em banco para o Mingau o tornará mais flexível abrindo novas possibilidades para o bot, pois poderá guardar informações de forma mais consistente, organizada e resume a programação hardcoded. Para substituír o sistema de arquivos .txt usaremos uma série de tabelas que relacionam cômodos, áreas, frases do bot e eventos que podem ocorrer. Pra começar temos a própria entidade Mingau, que é organizada da seguinte forma:  

<table align="center">
    <td align="center">
    Entidade Mingau
    </td>
  <tr>
    <td><img src="imagens\tab_mingau.png" width=500>
    </td>
  </tr>
</table>

> Nessa tabela mingau guarda informações importantes para a narrativa, como:  
>><b>id_do_bot:</b> Pode ser usado para gerenciar permissões nos canais;  
><b>último_cômodo:</b> Para o bot saber em qual cômodo ele esteve pela última vez, gerar mensagens de saída e continuar em caso de reiniciamento do bot;  
><b>humor:</b> Essa variável inteira será usada para determinar quais frasas podem ser selecionadas da tabela de frases;  ><b>interações:</b> Variáveis para calcular o momento em que mingau mudará de cômodo ou lugar;  
><b>usuário_preferido:</b> Indica qual é o  usuário por quem Mingau tem mais afinidade.  

</details>
<hr>

<details>
  <summary>Planejamento da memória do Cordano</summary> 
</details>
<hr>
