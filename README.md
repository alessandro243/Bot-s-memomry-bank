<h1 align='center'><b>Extensão da Cyber-República: Implementação de Memória em Banco de Dados</b></h1>

<br>
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
    <td><img src="imagens\tab_cômodos.png" width=200>
    </td>
    <td width=200>
    </td>
    <td><img src="imagens\tab_area.png" width=250>
    </td>
  </tr>
</table>
<br>
<hr>

### ⚙️ Tecnologias que serão utilizadas 

Linguagens, ferramentas e bibliotecas utilizadas no desenvolvimento do projeto:

* [Python:](https://www.python.org/) Linguagem base utilizada para integrar as tabelas ao projeto.
* [MySQL:](https://www.mysql.com/) SGBD escolhido por possuir APIs com documentação amplamente desenvolvida para integração com Python e oferecer suporte a múltiplas conexões.
* [pymysql:](https://pymysql.readthedocs.io/en/latest/) API utilizada para integrar o MySQL ao projeto.
* [sqlalchemy:](https://www.sqlalchemy.org/) API utilizada para integrar o MySQL ao projeto.
* [DBeaver:](https://dbeaver.io/) Programa utilizado para melhor visualização dos dados nas tabelas.
<br>
<br>
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
    <td><img src="imagens\tab_inventario.png" width=250>
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
    <td width=10>
    </td>
    <td><img src="imagens\tab_associação.png" width=150>
    </td>
  </tr>
</table>

> Observe que o que aconteceu aqui foi que Milka adicionou um novo usuário chamado Wandie Soul. Ele respondeu que seu ritmo de música preferido é J-pop, mas é relevante lembrar que representamos assim para melhor entendimento, e que não será adicionado com o nome do ritmo, e sim pelo ID do item cd_jpop, que está como 2. Através desse ID, vemos que Milka adicionou dois itens ao inventário no ID de Wandie Soul: a ração seca, em 3 unidades, e um CD de J-pop, em uma unidade. Então de dependendo da resposta, Milka adicionará um cd conforme os ritmos dispostos na tabela de ritmos:
> &nbsp;  

<table align="center">
    <td align="center">
    Entidade ritmo
    </td>
  <tr>
    <td><img src="imagens\tab_musics.png" width=150>
    </td>
  </tr>
</table>

> Importante lembrar, na hora da criação da tabela de rirmos que Milka usará os ritmos não deverão ter ids automáticos, pois para que tudo funcione corretamente, o ID do ritmo que constará na tabela deverá condizer com o ID do item cd do respectivo estilo musical na tabela item. Assim, quando Milka selecionar o estilo, poderá adicionar o item correto ao inventário no id do usuário.
> &nbsp;  

</details>

<hr>

<details>
  <summary>Planejamento da memória do Mingau</summary>
  <br>

> Para fins de exemplo, usaremos aqui o bot mingau, descrevendo como ele funcionará, mas o exposto será basicamente a dinâmica de funcionamento dos bots de ação aleatória, de modo geral.  
>O desenvolvimento de uma memória em banco para o Mingau tornará a dinâmica mais flexível, abrindo novas possibilidades para o bot, pois ele poderá guardar informações de forma mais consistente e organizada, além de reduzir a programação hardcoded. Para substituir o sistema de arquivos .txt, usaremos uma série de tabelas que relacionam cômodos, áreas, frases do bot e eventos que podem ocorrer. Para começar, temos a própria entidade Mingau, que é organizada da seguinte forma:  
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
    <td><img src="imagens\tab_evento.png" width=250>
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
  <summary>Planejamento da memória do Cordano</summary> 
  <br>

> Nessa configuração, Cordano não verificará mais um arquivo .json. Agora, para decidir como responder, o bot consulta a tabela interação, analisando se existe uma interação entre o bot e o usuário. Caso exista, Cordano saberá que não é a primeira interação e que não cabe um cumprimento. A entidade interação guarda os integrantes da interação a partir do momento em que ela acontece. No momento, planejamos guardar somente a data, mas talvez usemos o horário para uma segunda interação customizada. Essa tabela será limpa ao virar do dia para garantir que os cumprimentos aconteçam devidamente.  
> &nbsp;  
>  

<table align="center">
    <td align="center">
    Entidade interação
    </td>
  <tr>
    <td><img src="imagens\tab_interação.png" width=300>
    </td>
  </tr>
</table>  

> Da mesma forma, Cordano também passa a verificar a tabela inventário e a tabela associação antes de executar uma linha de diálogo. Ao fazer essa consulta, ele analisa se o usuário possui algum item com o qual o bot tenha relação; caso não tenha, segue sua linha de diálogo normal.  
>Se houver um item no inventário registrado no nome do usuário que tenha uma relação com o bot, ele executa uma fala especial e depois muda o estado da interação para o item correspondente ao ID do usuário na tabela associação.  
>Como funciona:  
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
    Considerando uma interação entre o usuário Wandie Soul e Cordano, podemos verificar na entidade inventário quais itens estão em posse desse usuário através do ID_user.
    </td>
</table>  

<table align="center">
    <td align="center">
    Entidade inventário
    </td>
    <td width=10>
    </td>
    <td align="center">
    Entidade item
    </td>
  <tr>
    <td><img src="imagens\tab_inventario2.png" width=250>
    <td><img width=10>
    <td><img src="imagens\tab_item2.png" width=450>
    </td>
  </tr>
</table>  

> Aqui observamos dois itens na posse do usuário Wandie Soul, a partir disso podemos usar o id dos itens para verificar se o bot tem uma interação especial com o item, na tabela associação. Agora um exemplo da entidade associação antes e depois da interação especial:  
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
    <td width=10></td>
    <td><img src="imagens\tab_associação_False.png" width=300></td>
    </td>
  </tr>
    <td align="center" width=300>
    Antes da interação especial, temos associação como verdadeira.
    </td>
    <td align="center" width=10>
    </td>
    <td align="cente" width=300>
    Após a interação especial, temos associação como falsa.
    </td>
</table>  

> Acerca das linhas de diálogo padrão de Cordano, fizemos uma entidade unicamente para as falas do NPC, nela temos diversas frases que ele usará em determinadas ocasiões. A tabela organiza as frases por bloco, então incialmente o personagem usará, como frases padrão, as que pertencem ao bloco 1. Para saber de qual bloco ele deve pegar suas frases padrão Cordano verificará a própria entidade Cordano, nela encontrará essa informação na coluna bloco_frase, o bloco que está escrito nessa tabela mudará conforme o avanço da narrativa e para controlar isso, não só Cordano, mas todos os personagens terão acesso à uma tabela de eventos globais, que lista com condição de verdadeiro ou falso para que saibam o que já aconteceu. Esse estado muda conforme os usuários interagem com a narrativa. Um exemplo de como funcionará essa dinâmica de narrativa e das tabelas envolvidas:  
> &nbsp;  
>  

<table align="center">
    <td align="center">
    Entidade interação
    </td>
    <td width=10></td>
    <td align="center">
    Entidade fala_cordano
    </td>
    <td width=10></td>
    <td align="center">
    Entidade evento
    </td>
  <tr>
    <td><img src="imagens\tab_cordano2.png" width=400>
    </td>
    <td width=10></td>
    <td><img src="imagens\tab_frases_cordano2.png" width=600>
    </td>
    <td width=10></td>
    <td><img src="imagens\tab_global_events.png" width=600>
    </td>
  </tr>
</table>  


> Aqui perceba que, inicialmente no bloco 1, Cordano está procurando por Milka e ele continuará repetindo esse conjundo de frases até que alguém encontre-a e diga isso, após essa interação o envento 2 em tab_evento terá seu estado mudado para True, o slot de diálogo de cordano mudará para 2 e ele usará novas linhas.  
>Perceba que aqui nós usamos o bot Cordano como exemplo, mas toda essa é a lógica aqui disposta é, de modo geral,como funcionarão os outros personagens/NPCs de ação não aleatória que comporão a totalidade da experiência.  
> &nbsp;  
>  

</details>

<hr>

<br>
<br>

<div align="center" width=700>
<p>

<p align="center">
<b>Sobre a idealização do projeto e seus objetivos na questão social</b>
</p>

Esse é o projeto de uma experiência multiplayer, de uma narrativa viva que se vive e se constroi em tempo real. Um dia pode ser um lugar tranquilo para quem só quer aproveitar uma boa vibe da ambientação, sem compromisso ou propósito. Mas também pode ser uma forma de se envolver em uma dinâmica gameficada viva que interage com você. A Cyber-república nasce de dois projetos reais. Um coletivo, que contando a narrativa de uma história, assume uma estética cyber-punk. Uma forma de expressar uma arte que não poderia ser transposta hoje, considerando toda a infra-estrutura envolvida no desenvolvimento de um jogo 3D dessa proporção. A engine que é o Discord foi para nós o ambiente perfeito para isso florescer porque já tem sistema de segurança, servidor, cadastro e comunicação entrincada.
Ela nasce, também, da sensação de que as redes sociais convencionais não são capazes de criar relações sociais verdadeiras e valorosas,estimulando pessoas a somar amigos em conversas rápidas com pessoas com as quais você raramente fala, mas chama de amigos. Nesse sentido, fundamos nosso lar, para ter um ambiente onde a conversa não é o objetivo, mas surge expontaneamente da sensação de estar em um ambiente acolhedor e imersivo sercado de pessoas que não sabem quem você é, mas foram capazes de sentir por esse lugar a mesma energia que você sentiu e tiveram a sensibilidade necessária para dar vida a algo que só existe em dois sentidos.
Como projeto de portifólio de um estudante, esse cantinho vai continuar crescendo enquanto eu puder aprender algo novo.
</p>
</div>