---
title: "[Auth 1/2] Proxy Reverso e como implementar"
project: false
layout: post
date: 2019-12-16 11:30:00 +0000
image: "/assets/images/nginx-logo.png"
headerImage: true
tag:
- haproxy
- http
- proxy
- nginx
- devops
- infraestrutura
category: blog
author: carlosdamazio
description: ''

---
Pessoal, tudo bem?

Então, postei no Instagram e dei uns _hints_ de que eu faria uma série de postagens sobre Autênticação de Serviços Existentes com 3 serviços: 1 Django Rest Framework (OAuth2) para autenticar o cliente na API, outro DRF para fazer a API da aplicação e o Vue.js para um serviço de front-end.

Essa vai ser a primeira parte da série de posts com relação à arquitetura de microsserviços, não especificamente sobre ela, mas sobre autenticação externa e como acoplar um sistema de autênticação a uma solução já existente no Django Rest Framework. Bem, mais do que explicar uma receita de bolo como de sempre, vou explicar uns conceitos...

## O que é uma Arquitetura de Microsserviços?

É uma arquitetura de solução de software que tem como diferencial a descentralização dos componentes. Ao invés de uma única solução ter componentes acoplados em uma única estrutura, estes componentes (autenticação, sistema de pagamento, dashboards, etc) são construídos em serviços pequenos e desacoplados, permitindo que estes serviços sejam independentes e escaláveis. Apesar de tudo, estes serviços ainda podem conversar entre sí através de protocolos (HTTP em grande parte).

![](/assets/images/microsserviço.png)

_Mas como assim "independentes e escaláveis"? Não seria mais fácil levantar duas instâncias da mesma aplicação?_

De forma alguma quero denegrir o que a arquitetura monolítica trouxe para gente. Pelo contrário, você tem que pensar muito bem sobre qual tipo de arquitetura usar, levando em consideração os requisitos e características que o seu projeto terá. Para fins de demonstração, vamos considerar um grande sistema que tem mais de 20 funcionalidades/módulos/submódulos/etc monolítico, como vamos garantir que:

* _A aplicação continue disponível caso um dos componentes falhe?_ Isso acontece muito quando temos uma aplicação MVC que está muito acoplada e confia demais na renderização no back-end;
* _Poderíamos fazer um deploy sem que haja indisponibilidade total da aplicação?_ Para cada atualização de uma funcionalidade ou módulo, precisamos fazer o deploy da aplicação inteira. Isso demanda **MUITO** tempo, dependendo do tamanho da aplicação;
* _Poderíamos reaproveitar funcionalidades de outros serviços sem que haja a necessidade de reescrita de código?_ Estes serviços deverão ser expostos às aplicações que desejam consumir a sua funcionalidade através das chamadas HTTP.
* _Haverá uma menor curva de aprendizado para novos desenvolvedores?_ Um sistema altamente acoplado acaba arruinando as noites de um novo dev, sendo que ele estaria encarregado de apenas um componente.

Então temos o seguinte:

* A gente não precisa punir uma aplicação inteira caso um dos seus serviços esteja com problemas;
* É desnecessário fazer deploy de 20 componentes cada vez que fazemos uma alteração em apenas um componentes. E podemos subir qualquer quantidade de instâncias separadas de quaisquer componentes havendo a necessidade para tal;
* Podemos garantir que uma pessoa não precise reinventar a roda.
* Podemos garantir que essa mesma pessoa se preocupe apenas com a complexidade do próprio módulo que ela esteja desenvolvendo;

_Beleza. Como vamos fazer que os serviços de uma aplicação ainda sejam alcançados? E como poderíamos escalar eles?_

Dai vou precisar explicar sobre um componente extremamente importante...

## Proxy reverso

![](/assets/images/proxy-reverso-1.png)

Proxy reverso é um componente na infraestrutura que tem como papel redirecionar requisições para um ou vários servidores web que contém serviços. Toda vez que um usuário solicitar algo ao back-end, a aplicação que ele acessa fará uma requisição ao proxy reverso, dentro do seu _namespace_, que, em seguida, fará uma requisição para o servidor web que serve este back-end em nome da aplicação _cliente_.

Vantagens de se utilizar um proxy reverso:

* Ele proverá cacheamento a um servidor back-end que se encontra em lentidão, assim diminuindo o tempo de resposta da aplicação devido a diminuição no tempo de resposta deste back-end;
* Pode prover balanceamento de carga entre várias instâncias de um back-end, fazendo assim uma aplicação de alto escopo e acesso performar com diferentes volumes de acesso, variando em qual instância o _cliente_ irá acessar;
* Não precisa registrar vários CNAME para se acessar os serviços (blog.empresa.com, suporte.empresa.com, app.empresa.com). Podemos simplesmente utilizar o _namespace_ do proxy reverso e fazer as requisições por eles mesmos (empresa.com/blog, empresa.com/app, empresa.com/suporte)

Existem várias ferramentas que podemos usar para implementar um proxy reverso: Apache, Nginx, HAProxy, Traefik, etc. Mas neste exemplo, estarei usando o Nginx para mostrar o uso desse componente.

## Sujando as mãos

![](https://logodownload.org/wp-content/uploads/2018/03/nginx-logo.png)

> Desejamos implementar um back-end mais resiliente, seguro e com balanceamento de carga. O que podemos fazer?

Devemos implementar um proxy reverso que, caso o cliente queira acessar o back-end da aplicação, poderá acessar uma das várias instâncias desse back-end **através** dele, e não diretamente nas instâncias, de acordo com a carga de cada uma delas.

Vamos criar 3 aplicações Flask distintas (para demonstrar o mecanismo de balanceamento de carga) que simularão o nosso back-end e iremos instanciar um Nginx com uma configuração de proxy reverso. Para isso, não precisamos de nada mais e nada menos do que o Docker e o Docker-Compose.

Primeiro, vamos criar as aplicações Flask e containerizar elas:

{% gist carlosdamazio/9e535b7314c4380c22a5b72ccb5035e3 %}

Para cada aplicação, mude o número dela no código fonte. Isso vai ser importante para demonstrar a capacidade de balanceamento (veremos isso no browser).

Crie um arquivo de _requirements.txt_. Ele será importante para baixarmos as nossas dependências dentro do container:

{% gist carlosdamazio/1da7daee70160d595733ac3867c26324 %}

Podemos criar um Dockerfile da seguinte maneira:

{% gist carlosdamazio/0950f6ccae52c9c94e63a8ca747e6148 %}

Por default, agora, todas as vezes que vamos rodar um container com a aplicação Flask, devemos estar atentos que estes vão rodar na porta 5000, e que vão precisar estar expostas na aplicação. Mas, nesse caso, como vamos usar o Nginx para acessar essa aplicação, não precisamos expor essas portas explicitamente pois não iremos acessar diretamente. Quem vai fazer esse trabalho é o Nginx.

E falando nele, na configuração de balanceador de carga, ele tem 6 métodos disponíveis para serem usados:

* Round Robin (default): todas as requisições que chegam no BC são distribuídas de forma igualitária;
* Least Connection: uma requisição é enviada para o servidor balanceado que tiver menos conexões ativas;
* IP Hash: uma requisição é enviada para um dos servidores balanceados de acordo com o endereço IP do requisitante (cliente). É calculado um hash dos últimos 3 octetos do endereço do IP, excelente para garantir que as requisições de um mesmo cliente cheguem ao mesmo servidor de destino, a menos que este esteja indisponível;
* Generic Hash: uma requisição é enviada para um dos servidores balanceados de acordo com uma chave, variável ou configuração definida pelo usuário. A chave pode ser pareada com o IP e porta do cliente ou a URI (recurso) do server;
* Least Time (apenas para o Nginx Plus): uma requisição é enviada para um dos servidores balanceados de acordo com a latência e o menor número de conexões ativas;
* Random: uma requisição é enviada para um dos servidores balanceados de forma aleatória usando um dos parâmetros de aleatoriedade (Least conn, Least time header, least time byte).

Neste caso, eu quero demonstrar que podemos acessar inúmeras instâncias de forma transparente. Mas no caso de uma aplicação real e distribuída, o usuário não deve e **NUNCA** deverá saber de qual instância ele está acessando. Para saber de mais detalhes desses métodos, acesse a [documentação](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) do Nginx. Para fins de demonstração, irei usar a configuração de _random_ normal:

{% gist carlosdamazio/1d3a74773b30f1d0a9c7a167892b520a %}

Depois, vamos criar um arquivo de Dockerfile, para criar um container do Nginx com as configurações acima:

{% gist carlosdamazio/336b24dbb68d39e46508b2873ff79a46 %}

Por último, vamos criar o arquivo do Docker Compose para levantar tudo o que montamos:

{% gist carlosdamazio/04488525742ed7930e771b9f307e2e95 %}

Com isso, rodamos no mesmo diretório o comando para subir o nosso exemplo:

    $ docker-compose up -d

E pronto! Acesse o http://localhost e fique pressionando o F5, veja o balanceador entrar em ação, trocando a cada F5 a aplicação que está acessando.

![](/assets/images/Peek 2019-12-15 00-15.gif)

## Considerações

Vimos a importância do proxy reverso como componente importante de uma arquitetura de microsserviços, mas estamos meio que escorando a superfície do que realmente é microsserviços. O objetivo aqui não era para demonstrar sobre o tema, mas sim para dar contexto e também para dar um _feeling_ do que teremos a seguir, pois é um tema que tem ligação sobre microsserviços. O próximo post será:

_Como integrar um sistema de autênticação OAuth2 em uma API feita no Django Rest Framework existente?_

Caso você tenha gostado deste post e se ele contribuiu para algo na vida, não esqueça de compartilhar o link ou mencionar que veio de cá! Meus contatos se encontram na página inicial do blog.

Fico feliz em ajudar.

:)