---
layout: post
title:  "Publicando WebService SOAP no Heroku"
date:   2016-05-06 14:29:42
author: Douglas Gabriel
categories: jekyll update
image: /img/posts/webservicesoap.jpg
description: Neste post, falo como publicar um serviço simples, desenvolvido em Java, no servidor cloud Heroku
keywords: tutorial
featured: true
category: tutorial
slider: true
index: true
categoryslider: true
---

Saudações,

Neste post pretendo falar um pouco sobre serviços e como disponibilizar um utilizando o servidor cloud [Heroku].

> Um serviço, basicamente, trata-se de um componente que disponibiliza uma funcionalidade de forma independente, ou seja, não dependendo do estado de outros, podendo ser acessado, na maioria das vezes, através de um Web Service de forma que, diferentes aplicações, independentemente de tecnologia, consigam interagir com o mesmo.

> Para isto, deve disponibilizar uma interface que defina as possíveis operações suportadas, tornando viável esta interação.
Web Services são, portanto, aplicações que disponibilizam um serviço ou um conjunto deles, conectados através de um barramento de serviços, acessados através de suas respectivas interfaces.

> Para este exemplo iremos utilizar as seguintes tecnologias:
- Java;
- JAX-WS;
- Maven;
- Git.

>O ambiente de desenvolvimento utilizado foi o sistema Ubuntu 15.10 em conjunto com a ferramenta Netbeans.

> O código completo deste exemplo pode ser encontrado [aqui][repository]

Com os conceitos básicos em mente, e certo que já providenciaram vossas contas no Heroku, vamos começar o nosso guia.

Primeiro desenvolveremos um serviço simples, utilizando a solução Java, JAX-WS, com um único método "Hello World":

<script src="https://gist.github.com/douglasgabriel/3c82543fc416e76763027ba0433e9cc8.js"></script>

Neste código apenas um detalhe, a anotação @WebService indica que esta classe corresponde à um webservice, os atributos especificados são obrigatórios, sendo que *endpointInterface* corresponde ao caminho totalmente qualificado da classe (pacote + nome da classe) e *serviceName* corresponde ao nome da classe.

Assim, nosso WebService simples está implementado! Agora é só disponibilizá-lo:

<script src="https://gist.github.com/douglasgabriel/08fce72e5747b23bbb8ed22d923e5386.js"></script>

Executando a classe acima, o serviço estará disponível no localhost na porta 8080, na URI /service, sendo que tudo isto fica a seu critério, porém, na hora de acessar há um detalhe, é necessário adicionar ?wsdl ao final da URL:

```http://localhost:8080/service?wsdl```

Portanto, neste endereço estará disponibilizado o WSDL do serviço, que nada mais é do que um XML que descreve o acesso ao serviço, seu métodos e entre outras informações, funcionando, assim, como a sua interface.

Com o WebService pronto e funcionando, é hora de trabalharmos com o Heroku.

Com sua conta já criada, para este tutorial iremos utilizar a ferramenta disponibilizada pelo Heroku, o Heroku Toolbelt, que facilitará bastante nossa vida.

Com a ferramenta já instalada na sua máquina, com o terminal, navegue até a pasta do seu projeto e digite:

``` heroku login ```

Fornecendo suas credenciais, termine a operação.

Feito isto, o proximo passo é o seguinte comando tornar a pasta do projeto gerenciada pelo git com o comando:

```git init```

E assim, criar uma aplicação Heroku com o seguinte comando:

```heroku apps:create <nome-da-sua-aplicacao>```

Com este comando, o Heroku irá disponibilizar uma URL válida na Internet para o seu projeto, e um repositório Git, o qual o seu projeto deverá ser enviado para que seja feito o deploy automático.

Antes de qualquer coisa, deveremos fazer algumas alterações na nossa classe principal, uma vez que não mais iremos publicar nosso serviço localmente. Vejamos:

<script src="https://gist.github.com/douglasgabriel/cf1b8b86badbe3a1bf2723fcbf26b4b3.js"></script>

No código acima, é possível vermos alguns detalhes:

Primeiramente, a forma como definimos a porta do nosso serviço mudou, isso se dá porque estamos delegando ao Heroku a responsabilidade de informar-nos a porta ao qual a nossa aplicação está sendo disponibilizada, para isto, nós precisamos apenas recuperar uma variável de ambiente com o comando ``` System.getenv("PORT") ```.

Além disto, o host também mudou, setamos para 0.0.0.0, esta prática, no contexto de servidores, indica que nossa aplicação ficará disponível em qualquer endereço disponibilizado, basicamente, delegamos mais uma vez ao Heroku a responsabilidade de configurar o host.

E por fim, definimos mais uma vez a URI onde o nosso serviço ficará disponível com o valor "/service".

Após configurarmos a classe principal, agora adicionaremos o plugin Apache Maven Assembly, no nosso arquivo pom.xml, com o intuito de tornar nossa aplicação executável, informando nossa classe principal:

<script src="https://gist.github.com/douglasgabriel/33b4f34c5789a7cff00784116d0e769b.js"></script>

O único detalhe é a adição do caminho totalmente qualificado da sua classe principal na propriedade *mainClass*.

Feito isso, agora é nessário a criação de um arquivo chamado *Procfile* na raiz do projeto, onde iremos informar o Heroku a forma de execução da nossa aplicação. No terminal, digite:

```nano Procfile```

Dentro do arquivo coloque

```web: java -jar <caminho da aplicação>```

No meu caso, o caminho da aplicação é o seguinte:

```target/sample-service-heroku-1.0-SNAPSHOT-jar-with-dependencies.jar```

Feitas estas configurações, é hora de fazer o deploy da nossa aplicação. Para isto, precisaremos simplesmente utilizar comandos git (é possível encontrar um tutorial git [aqui][git]):

Com o terminal no diretório da aplicação, rode o seguinte comando:

``` git add . ; git commit -m "deploying" ; git push heroku master ```

Pronto, com isto a sua aplicação já foi construída no servidor Heroku, porém ainda não deve estar disponível. Para finalmente disponibilizar sua aplicação, acesse o seu painel de controle no site do Heroku, localize e clique em sua aplicação, na aba *Overview* localize e clique na opção *Configure Dynos*, você irá se deparar com o seguinte elemento:

<img src="/img/posts/heroku-dynos.png">

Clique no lápis para editar as configurações, em seguida clique no *swipe* e confirme as alterações.

Pronto! Com isto sua aplicação agora encontra-se disponível e para acessá-la localize e clique, no canto superior direito o botão *open app*, uma nova guia será aberta, não se esqueça de adicionar a URI do seu serviço ao endereço da aplicação, no caso ```/service?wsdl```

Agora, vamos testar o serviço consumindo-o através de uma outra aplicação Java.

Para isto, será necessário definir a interface que corresponde ao serviço, assim como vemos no código abaixo:

<script src="https://gist.github.com/douglasgabriel/fc93a957280c60bd0a71c64135c5a31a.js"></script>

Mais uma vez utilizamos a anotação @WebService, desta vez para especificar as informações de qual serviço estaremos consumindo com esta interface, as informações passadas nos atributos da anotação podem ser recuperadas no XML gerado, na propriedade *definitions*.

Precisamos também de um método com a mesma assinatura especificada pelo serviço, no caso ```public String hello(String name)```

Por fim, para recuperarmos o serviço é só adotar o procedimento visto abaixo:

<script src="https://gist.github.com/douglasgabriel/dec72f8575018dd5415b436e8f7d14b1.js"></script>

Simples, não é?

Mais alguns detalhes:

> O Heroku disponibiliza os logs da aplicação através do comando ```heroku logs```

> O Heroku automaticamente já disponibiliza um banco de dados para a sua aplicação, é possível recuperar as informações de como acessá-lo pelo comando ```heroku config:get DATABASE_URL```

Até a próxima!

[heroku]:https://www.heroku.com/
[repository]:https://github.com/douglasgabriel/sample-service-heroku
[git]:http://douglasgabriel.github.io/tutorial/2015/04/03/Utilizando-Git.html
