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