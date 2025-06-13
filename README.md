<h1 align='center' style ="font-size: 18px"><b>Extensão do Projeto República: Implementação de Memória em Banco de Dados</b></h1>

Como bem vimos, os cômodos de nossa república comunicam-se através de arquivos de texto. Isso funciona bem com as consultas, que na maior parte das vezes são rápidas e simples, usando apenas um dado simples, como dia, segundos ou um dado booleano. Só que isso não serve para estruturas de dados mais complexas, extensas e não uniformes, como listas de dicionários grandes, o que, da forma que fazemos atualmente, é um problema quando consideramos desenvolver uma memória consistente para um NPC, pois não é escalável e não oferece segurança nos dados.

Do modo que está, você só passa a ter algum tipo de registro em memória a partir do momento em que fala com o Cordano no bar. Ele então adiciona você a um registro de usuários em um arquivo .json. Nesse arquivo haverá informações como nome do usuário, se você está interagindo pela primeira vez e sua lista de itens. A ideia era que cada bot usasse esse .json, onde teria, para cada item do usuário, uma lista de bots que interagem com esse item e, após a interação, ele apagaria seu nome dessa lista, inutilizando determinada linha de diálogo.
Mas isso passa a não funcionar tão bem quando consideramos um número grande de usuários e vários itens, pois o arquivo ficaria cada vez mais pesado, confuso e não seria uniforme. Ainda devemos considerar que determinado bot pode acabar alterando uma informação que outro bot usaria, causando erros de narrativa.

Dessa forma, nós planejamos implementar uma rede de tabelas para cada bot, estabelecendo uma memória robusta e persistente, com suporte a várias conexões para cada personagem. Então, ao entrar no servidor pela primeira vez, o usuário será recepcionado por um bot administrativo, que lhe fará perguntas para otimizar a experiência do usuário — perguntas como nome, estilo de música preferido. Ele então registrará as informações na tabela de usuários, com um inventário para itens. Dessa forma, os bots poderão acessar esse inventário para gerenciar suas próprias frases e ações.

Primeiramente, teremos uma tabela de cômodos, com a qual todos os bots tomarão ciência dos cômodos existentes na república, e também uma tabela de áreas, representando partes dos cômodos. A princípio, a tabela de áreas é mais usada pelo nosso mascote Mingau, mas, assim como a de cômodos, poderá ser usada por todos os bots.

<table align='center'>
    <td align='center'>
    Entidade Cômodo
    </td>
    </td>
    <td>
    <td align='center'>
    Entidade Área
    </td>
  <tr>
    <td><img src="imagens\tab_cômodos.png" width=250>
    </td>
    <td width=200>
    </td>
    <td><img src="imagens\tab_area.png" width=300>
    </td>
  </tr>
</table>

<hr>

### ⚙️ Tecnologias que serão utilizadas 

Linguagens, ferramentas e bibliotecas utilizadas no desenvolvimento do projeto:

* [Python:](https://www.python.org/) Linguagem base utilizada para integrar as tabelas ao projeto.
* [MySQL:](https://www.mysql.com/) SGBD escolhido por possuir APIs com documentação amplamente desenvolvida para integração com Python e oferecer suporte a múltiplas conexões.
* [pymysql:](https://pymysql.readthedocs.io/en/latest/) API utilizada para integrar o MySQL ao projeto.
* [sqlalchemy:](https://www.sqlalchemy.org/) API utilizada para integrar o MySQL ao projeto.
* [DBeaver:](https://dbeaver.io/) Programa utilizado para melhor visualização dos dados nas tabelas.
<hr>

<details>
  <summary>Planejamento de cadastro de usuários</summary>
  <br>

> Uma lógica de interação entre bots e usuários por meio da análise das tabelas abre várias possibilidades em termos de escalabilidade. Porém, para que a dinâmica não fique comprometida, ainda é preciso otimizar a questão do registro de novos usuários. Como já vimos, o registro do usuário acontece quando ele fala com o Cordano, então, antes disso, ele não terá nenhum registro ou inventário, não podendo, assim, pegar quaisquer itens. A ideia é transferir a responsabilidade do registro dos usuários do Cordano para a administração, com quem ele interagirá assim que chegar.  
> &nbsp;  
> Quando um novo user se conectar ao servidor, ele ou ela será recepcionado por Milka, que fará uma breve pesquisa colhendo dados que usará no registro. Após o pequeno questionário, Milka adicionará o user ao registro de membros do servidor da seguinte forma:  
> &nbsp;  

<table align="center">
    <td align="center">
    Entidade user
    </td>
  <tr>
    <td><img src="imagens\tab_users.png" width=400>
    </td>
  </tr>
</table>

> Com base nas respostas do usuário, Milka escreverá seu ritmo preferido e sua afinidade com Mingau, que na maioria dos casos começa em zero. Logo depois, ela adicionará um item ao inventário no ID da pessoa. Segue um exemplo da entidade Inventário:  
> &nbsp;    

<table align="center">
    <td align="center">
    Entidade inventário
    </td>
  <tr>
    <td><img src="imagens\tab_inventario.png" width=300>
    </td>
  </tr>
</table>

> Através do ID dos itens dispostos na tabela Inventário, os bots poderão acessar a tabela Item para ver as descrições ou a tabela Associação para verificar sua interação com eles, caso precisem considerá-los antes da execução de um evento pontual. Ao encontrar o item que procura para executar determinado evento, o bot então o executa e muda a área com seu nome para null. Dessa forma, ele não interagirá mais com esse item, perdendo essa linha de diálogo com esse usuário.  
> &nbsp;  

<table align='center'>
    <td align='center'>
    Entidade item
    </td>
    </td>
    <td>
    <td align='center'>
    Entidade associação
    </td>
  <tr>
    <td><img src="imagens\tab_item.png" width=400>
    </td>
    <td width=200>
    </td>
    <td><img src="imagens\tab_associação.png" width=200>
    </td>
  </tr>
</table>

> Observe que o que aconteceu aqui foi que Milka adicionou um novo usuário chamado Wandie Soul. Ele respondeu que seu ritmo de música preferido é J-pop, mas é importante lembrar que representamos assim para melhor entendimento, e que não será adicionado com o nome do ritmo, e sim pelo ID do item cd_jpop, que está como 2. Através desse ID, vemos que Milka adicionou dois itens ao inventário no ID de Wandie Soul: a ração seca, em 3 unidades, e um CD de J-pop, em uma unidade.  
> &nbsp;  

</details>


</details>

<hr>

<details>
  <summary>Planejamento da memória do Mingau</summary>
  <br>

> O desenvolvimento de uma memória em banco para o Mingau tornará a dinâmica mais flexível, abrindo novas possibilidades para o bot, pois ele poderá guardar informações de forma mais consistente e organizada, além de reduzir a programação hardcoded. Para substituir o sistema de arquivos .txt, usaremos uma série de tabelas que relacionam cômodos, áreas, frases do bot e eventos que podem ocorrer. Para começar, temos a própria entidade Mingau, que é organizada da seguinte forma:  
> &nbsp;  

<table align="center">
    <td align="center">
    Entidade Mingau
    </td>
  <tr>
    <td><img src="imagens\tab_mingau.png" width=500>
    </td>
  </tr>
</table>

>Nessa tabela mingau guarda informações importantes para a narrativa, como:  
>&nbsp;&nbsp;&nbsp;<b>id_do_bot:</b> Pode ser usado para gerenciar permissões nos canais;  
>&nbsp;&nbsp;&nbsp;<b>id_último_cômodo:</b> Para o bot saber em qual cômodo ele esteve pela última vez, gerar mensagens de saída e &nbsp;&nbsp;&nbsp;continuar em caso de reiniciamento do bot;  
>&nbsp;&nbsp;&nbsp;<b>id_última_área:</b> Para o bot saber em qual área ele esteve pela última vez, gerar mensagens de saída e &nbsp;&nbsp;&nbsp;continuar em caso de reiniciamento do bot;  
>&nbsp;&nbsp;&nbsp;<b>humor:</b> Essa variável inteira será usada para determinar quais ações podem ser selecionadas da tabela de &nbsp;&nbsp;&nbsp;ações;  
>&nbsp;&nbsp;&nbsp;<b>interações:</b> Variável usada para calcular o momento em que mingau mudará de cômodo ou área;  
>&nbsp;&nbsp;&nbsp;<b>usuário_preferido:</b> Indica qual é o usuário por quem Mingau tem mais afinidade.  
>  
> Através da tabela de cômodos, Mingau tomará ciência de por quais cômodos poderá transitar, emitindo mensagens de transição e atualizando seu estado de último cômodo.  
> A segunda entidade será uma tabela de ações que Mingau poderá executar. Ela guardará frases categorizadas por humor e por área. Assim, quando o comando !mingau for acionado, ele poderá fazer as verificações e tomar ações de acordo com seu humor e com a localização do cômodo em que se encontra:  
> &nbsp;  

<table align="center">
    <td align="center">
    Entidade ação
    </td>
  <tr>
    <td><img src="imagens\tab_frase.png" width=500>
    </td>
  </tr>
</table>

> &nbsp;&nbsp;&nbsp;<b>condição:</b> Variável decisiva para a escolha de frases do bot;  
> &nbsp;&nbsp;&nbsp;<b>valor:</b> Define o valor que afetará o humor de Mingau, a ação e seu valor serão adicionados a tabela  
>&nbsp;&nbsp;&nbsp;eventos e essa será usada no cálculo de humor ao fim da interação.  
>  
> Dessa forma, a partir do uso do comando !mingau, a operação passa a seguir o seguinte rumo:  
>  
> 1 - Consulta a entidade Mingau para pegar informações como cômodo, área, número de interações e humor, que começa como neutro. Por exemplo, se o humor do Mingau for 0 (neutro), buscaremos ações com condição igual a zero ou NULL; caso seja 1 (positivo), buscaremos ações com condição > 0 ou NULL; e se for -1 (negativo), buscaremos ações com condição < 0 ou NULL. Ações com condição NULL podem ocorrer em quaisquer estados de humor.  
> 2 - Em seguida, verificamos a tabela de frases, selecionando as que condizem com sua área e estado de humor.  
> 3 - Selecionamos aleatoriamente uma frase, salvando o texto e o valor que ela agrega ao estado de humor. Esse valor vem da coluna "valor" da tabela de ações, de onde tiramos o número que será somado ao humor de Mingau. Assim, se a frase tiver um valor positivo, o humor de Mingau ficará mais elevado.  
> 4 - Emitimos o texto da mensagem no canal e atualizamos o humor de Mingau. O humor é calculado com base na coluna "efeito_humor" da tabela de eventos. Nessa tabela, registramos as ações que já foram executadas e como elas afetam o humor. Com base na coluna "efeito_humor", fazemos o somatório e atualizamos o estado de humor ao final de cada ação.  
> 5 - Agora, Mingau estará pronto para o próximo comando.  
>  
> Segue a tabela de eventos:  
> &nbsp;  
> 

<table align="center">
    <td align="center">
    Entidade evento
    </td>
  <tr>
    <td><img src="imagens\tab_evento.png" width=300>
    </td>
  </tr>
</table>

> Acima, vemos um exemplo da tabela de eventos e, com base na coluna efeito_humor, ao fim da operação recalcularemos o humor de Mingau. Por exemplo: 0 - 1 + 0 resulta em um total de -1, indicando um humor negativo. Esse valor definirá as próximas ações de Mingau.  
> &nbsp;  
> Mingau também terá dois novos comandos: !alimentar e !brincar. Esses comandos adicionarão à tabela de eventos um registro com um efeito_humor positivo, afetando o humor de Mingau. Além disso, eles acrescentarão pontos a um contador chamado pontos_mingau, localizado na tabela do próprio usuário. No início do próximo dia, Mingau atualizará sua afinidade com o usuário com base nesse contador.  
> &nbsp;  
> 
</details>
<hr>

<details>
  <summary>Ação e memória da Milka</summary> 
</details>
<hr>

<details>
  <summary>Planejamento da memória do Cordano</summary> 
  <br>

> Nessa configuração, Cordano não verificará mais um arquivo .json, agora, para decidir como responder, o bot consulta a tabela interação, analisando se existe e a interação entre o bot e o usuário. Existindo, Cordano saberá que não é a primeira interação e que não cabe um cumprimento. Essa entidade interação guarda os integrantes da interação a partir do momento em que ela acontece, no momento, planejamos guardar somente a data, mas talvez usemos o horário para uma segunda interação customizada. Ela será limpa ao virar do dia para garantir que os cumprimentos aconteçam devidamente.  
> &nbsp;  
>  

<table align="center">
    <td align="center">
    Entidade interação
    </td>
  <tr>
    <td><img src="imagens\tab_interação.png" width=400>
    </td>
  </tr>
</table>  

> Da mesma forma Cordano também passa a verificar a tabela inventário e a outra tabela item antes de executar uma linha de diálogo, fazendo a consulta ele analisa se o usuário tem um item com o qual o bot se relacione, caso não tenha, ele segue sua linha de dialogo normal. Se houver um item no invetário registrado no nome do usuário e que tenha uma relação com o bot, ele executa uma fala especial e depois muda o estado da interação com item que está no id do usuário na tabela associação. Como funciona:  
> &nbsp;  
>  

<table align="center">
    <td align="center">
    Entidade user
    </td>
  <tr>
    <td align="center"><img src="imagens\tab_users.png" width=400>
    </td>
  </tr>
  <td align="center" width=500>
    Considerando uma interação entre o usuário Wandie Soul e Cordan, podemos verificar na entidade inventário quais itens estão em posse desse usuário através do ID_user.
    </td>
</table>  

<table align="center">
    <td align="center">
    Entidade inventário
    </td>
  <tr>
    <td><img src="imagens\tab_inventario.png" width=300>
    </td>
  </tr>
</table>  

> Aqui observamos dois itens na posse do usuário Wandie Soul e a partir disso podemos usar o id dos itens para verificar se o bot tem uma interação especial com o item na tabela associação. Agora um exemplo da entidade associação antes e depois da interação especial:  
> &nbsp;  
>  

<table align="center">
    <td align="center">
    Entidade associação
    </td>
    <td align="center">
    </td>
    <td align="center">
    Entidade associação
    </td>
  <tr>
    <td><img src="imagens\tab_associação_True.png" width=300></td>
    <td width=300></td>
    <td><img src="imagens\tab_associação_False.png" width=300></td>
    </td>
  </tr>
    <td align="center" width=400>
    Antes da interação especial, temos associação como verdadeira.
    </td>
    <td align="center" width=500>
    </td>
    <td align="center" width=400>
    Após a interação especial, temos associação como falsa.
    </td>
</table>  

</details>
<hr>
