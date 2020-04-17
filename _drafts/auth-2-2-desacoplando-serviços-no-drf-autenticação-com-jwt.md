---
title: "[Auth 2/2] Desacoplando serviços no DRF: autenticação com JWT."
project: false
layout: post
date: 2020-04-18 03:00:00 +0000
image: "/assets/images/django-logo-negative.png"
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

## Disclaimer rápido

Lembro a vocês que micro serviços, o seu objetivo é de desacoplar a estrutura, e isso inclui até mesmo TODA a estrutura que esse serviço necessita, incluindo o seu BD, ao contrário do que eu sugeri na imagem do post passado. E eu relembro pra vocês que o seu uso tem que ser muito bem pensado, pois não é em toda solução em que se é adequado aplicar essa arquitetura. 

Um blog simples não é um caso necessário de se aplicar micro serviços, e nem uma aplicação de lista de _TO-DO_. Toda utilização e consideração de uso de uma arquitetura tem que ter uma explicação e cabe a gente mesmo decidir o que usar e como usar, mas parte do bom senso de pesquisa e de uso da ciência.

Vamos fazer um _refresh_ sobre o modelo de autenticação do Django Rest Framework.