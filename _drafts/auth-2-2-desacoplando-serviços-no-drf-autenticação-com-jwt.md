---
title: "[Auth 2/2] Desacoplando serviços no DRF: autenticação com JWT."
project: false
layout: post
date: 2020-04-18 03:00:00 +0000
image: "/assets/images/logo.png"
headerImage: true
tag:
- jwt
- drf
- django
- web
- python
- dev
category: blog
author: carlosdamazio
description: ''

---
Galera, primeiramente, malz pelo sumiço! kkkkk

Estava muito "atarefado", tanto com coisas de trabalho, quanto no aspecto pessoal. Passei por uns apertos, e com o [COVID-19](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=20&cad=rja&uact=8&ved=2ahUKEwiBx8ybivDoAhU0CrkGHXAnCVwQFjATegQIFRAB&url=https%3A%2F%2Fpt.wikipedia.org%2Fwiki%2FCOVID-19&usg=AOvVaw2Oe8oOkuGX4jcR8uCAiGtD), não seria diferente. Para que possamos passar bem por essa crise, não há nada melhor do que fazer coisas que nos façam crescer, sendo escrever, aprender, exercitar, conversar com os nossos entes queridos (mesmo que virtualmente). Então esta é uma boa oportunidade para gente crescer, tanto pra mim que escreve por aqui, quanto pra você que pode aprender alguma coisa nisso tudo.

Sem mais delongas, eu prometi desde a última vez que escrevi (12/2019) uma postagem sobre desacoplar serviços de API no Django DRF. Eu aprendi muita coisa desde a última postagem, então pode ser que muitas coisas que eu escrevi estejam equivocadas ou erradas mesmo, então eu peço desculpas pelos erros e que possamos aprender juntos.

Sendo mais específico, vou tentar provar que é possível se criar um serviço de autenticação no DRF para que se possa utilizar um outro serviço. A única incerteza do experimento é de que o quão bom é o Django em fazer desacoplamento de serviços de autenticação, sendo que isso é bastante acoplado e enraizado no framework com a questão do _request.user_, ou com _AnonymousUser_, ou com qualquer coisa que é relacionada a usuário no framework.

## Disclaimer rápido

Lembro a vocês que micro serviços, UM dos seus objetivos é de desacoplar a estrutura, e isso inclui até mesmo TODA a estrutura que esse serviço necessita, incluindo o seu banco de dados, ao contrário do que eu sugeri na imagem do post passado. E eu relembro pra vocês que o seu uso tem que ser muito bem pensado, pois não é em toda solução em que se é adequado aplicar essa arquitetura.

Um blog simples não é um caso necessário de se aplicar micro serviços, e nem uma aplicação de lista de _TO-DO_. Toda utilização e consideração de uso de uma arquitetura tem que ter uma explicação e cabe a gente mesmo decidir o que usar e como usar, mas parte do bom senso de pesquisa e de uso da ciência.

Digo e repito: Prova de conceito **NÃO É PRODUCTION READY**. São ideias do que podemos fazer com uma determinada tecnologia, não é pra fazer CTRL+C e CTRL+V, seja responsável!

Vamos fazer um _refresh_ sobre os modelos de autenticação do Django Rest Framework.

## Como é determinado a autenticação no DRF?

Dentro das configurações de um projeto DRF a gente costuma passar uma lista de classes de autenticação em um item de configuração chamado _DEFAULT AUTHENTICATION CLASSES._ Para cada classe, o Django tenta autenticar com cada uma dessas classes e vai aplicar um valor para o _request.user_ e _request.auth_ usando o valor da primeira classe que autenticar o usuário com sucesso.

Caso não ocorra autenticação alguma, o _request.user_ será definido como _AnonymousUser_, que é uma instância de um usuário que não possui relação alguma com o serviço.

Temos uma lista de tipos de autenticação que são providos, tanto nativamente como adquiridos de terceiros, como:

* BasicAuthentication;
* TokenAuthentication;
* SessionAuthentication;
* RemoteUserAuthentication;
* OAuth;
* JWTAuthentication.

Não existe uma melhor autenticação, afinal, grande parte delas estão descritas em RFCs (_Request for Comments_) que são documentos de especificações técnicas de tecnologia escritas por tanto indivíduos como entidades. Mas o que esteve sempre em evidência, na comunidade de desenvolvimento web, é a utilização do JWT. Creio que seja pelo grande uso do JavaScript na comunidade, eu não sei qual a correlação no momento (talvez eu edite o post mais tarde). Mas seu uso é válido, provado pela especificação na [RFC 7519](https://tools.ietf.org/html/rfc7519) prova a sua veracidada.

## JWT

JWT (JSON Web Token) é um padrão para envio de "claims" em espaços beeeem restritos. O que isso significa? É um token que, ao mesmo tempo que verifica a identidade de quem a possui, também pode repassar dados que podemos inserir, além de podermos prover clientes _stateless_. Claro, aparentemente isso não é novidade, mas promete trazer simplicidade na hora de fazer isso.

Vamos analisar um JWT:

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1NDk4NzQ5ODc5OCIsIm5hbWUiOiJDYXJsb3MgRGFtw6F6aW8iLCJpYXQiOjQ5ODQ5ODc5ODE2LCJtc2ciOiJFdSBzZWkgcXVlIHZvY8OqIGVzdMOhIGxlbmRvIGlzc28uIn0.LLZkJu000yUAfuE91wf69eajjL7uwblcZRVpFRHDVCQ

Meio complexo? Cheio de lixo? Não se parece com nada de JSON? Ele está codificado em Base64url Vamos quebrar esse token em 3 pontos ".":

    <header>.<payload>.<signature>

O que significa cada uma dessas estruturas? Se a gente decodificar cada uma dessas sessões, a gente tem:

* Header: É o metadado do JWT, ele fala do que a informação como um todo se trata e como a sua _signature_ está codificada (qual algoritmo de codificação está sendo utilizado):

      {
        "alg": "HS256",
        "typ": "JWT"
      }
* Payload: possuí alguns dados de usuário, também pode conter alguns metadados do token como sua expiração:

      {
        "sub": "54987498798",
        "name": "Carlos Damázio",
        "iat": 49849879816
      }
* Signature: é a assinatura pública do JWT, que é composta pelo "header.payload" e o _segredinho_ do server:

      HMACSHA256(
        base64UrlEncode(header) + "." +
        base64UrlEncode(payload),
        segredinho
      )

Sendo assim, temos o JWT dessa forma:

    Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1NDk4NzQ5ODc5OCIsIm5hbWUiOiJDYXJsb3MgRGFtw6F6aW8iLCJpYXQiOjQ5ODQ5ODc5ODE2LCJtc2ciOiJFdSBzZWkgcXVlIHZvY8OqIGVzdMOhIGxlbmRvIGlzc28uIn0.LLZkJu000yUAfuE91wf69eajjL7uwblcZRVpFRHDVCQ

Agora, sabendo como o JWT funciona, vamos traçar a nossa estratégia para implementá-lo em código.

## Definição de arquitetura

Normalmente, teríamos uma aplicação cliente que faria acesso aos serviços, mas o objetivo é mostrar a nível de serviço e de requisição. Temos 2 serviços, um será o serviço de autenticação e outro será um serviço qualquer que dependerá da autênticação.

* O serviço de autenticação ficará responsável por emitir os tokens para os usuários cadastrados dentro do sistema. Ao fazer o login, o usuário recebe o JWT. Normalmente, a aplicação cliente fica no encargo de guardar este token, normalmente em um Storage. Em nosso caso, vamos guardar na mão pra mostrar passo a passo como que o desacoplamento de serviços poderia funcionar no Django;
* Cada serviço vai ter o seu próprio banco de dados;
* Teremos um serviço fictício de transações bancárias. Ele receberá um JSON de transação. Mas somente receber esse JSON não é o suficiente, ele precisa receber um JWT no header de Authorization. E nesse serviço, será feito a sua validação, antes de realizar qualquer operação. Para fazer isso, precisaremos escrever um middleware que receberá essa requisição para ser tratado esse token a parte. 

Esse é um dos problemas de se fazer desacoplamento de autenticação no Django, pois a gente perde uma base de usuários e também os middlewares de autenticação pra facilitar a nossa vida! Mas é possível de se fazer. _Shit happens._

![](https://media.giphy.com/media/wdNS4Q0Kkwkda/giphy.gif)

Bora parar de _pipipi popopo_ e ver a bagaça funfando, hein?!

## Codando o serviço de autenticação

Eu estou utilizando o Kubuntu 18.04 LTS com a versão do Python 3.8.2. A gente vai criar dois projetos, um chamado _auth_ e outro chamado _transaction_, e para cada um deles, iremos inicial um ambiente virtual do Python (virtualenv) utilizando a opção -m invocando o módulo de _venv_:

    $ mkdir auth transaction
    $ cd auth && python3.8 -m venv .venv; cd ../transaction && python3.8 -m venv .venv && cd ..

Criados ambos os diretórios com as suas venvs, vamos iniciar o nosso primeiro serviço: o de autenticação. Vamos entrar no diretório, ativar a _venv_ e logo em seguida instalar o Django, o DRF, o DRF SimpleJWT plugin e o psycopg2 para usar o banco de dados PostgreSQL. É importante fazer o freeze dos requisitos do projeto como boa prática de mantermos as dependências em cheque:

    $ cd auth
    $ source .venv/bin/activate
    (.venv) $ pip install django djangorestframework djangorestframework-simplejwt psycopg2
    (.venv) $ pip freeze > requirements.txt

Para iniciar um projeto Django, basta rodar o seguinte comando:

    (.venv) $ django-admin startproject auth .

Ele vai criar a estrutura de diretórios certinha para podermos iniciar a desenvolver o nosso serviço django. Sua estrutura de diretórios deverá estar assim:

    .
    ├── auth
    │   ├── asgi.py
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── manage.py
    └── requirements.txt

Vamos por partes. Nós temos um projeto padrão Django normal, sem ser o Django Rest Framework. Teremos que adicionar mais algumas coisinhas pra gente deixar do jeito que é para ser. Em _settings.py_, vamos adicionar o app do DRF em INSTALLED_APPS e a classe de autenticação JWT nas configurações do REST_FRAMEWORK:

    INSTALLED_APPS = [
    	...,
        'rest_framework',
    ]
    
    ...
    
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework_simplejwt.authentication.JWTAuthentication',
        )
    }
    
    ...

Perfeito. Em urls.py no root da aplicação, vamos adicionar 3 endpoints dentro do URL patterns com as views do JWT. O código ficaria assim:

{% gist carlosdamazio/c03aa3f489eb1b568e857ae0c7337ce9 %}