---
title: "[Autenticação de Serviços Existentes] Calmaria antes da tempestade: Proxy
  Reverso e como implementar um"
project: false
layout: post
date: 2019-12-12 03:00:00 +0000
image: ''
headerImage: false
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

Então, postei no Instagram e dei uns _hints_ de que eu faria uma série de postagens sobre Autênticação de Serviços Existentes com 3 serviços: 1 Django Rest Framework (OAuth2) para autenticar, outro DRF para fazer o back-end da aplicação e o Vue.js para um serviço de front-end. 

Vai ser uma série de post com relação à arquitetura de microsserviços, não especificamente sobre ela, mas sobre autenticação externa e como acoplar ela em uma solução já existente no Django Rest Framework. Bem, mais do que explicar uma receita de bolo como de sempre, vou explicar uns conceitos...

## O que é uma Arquitetura de Microsserviços?

É uma arquitetura de solução de software que tem como diferencial a descentralização dos componentes. Ao invés de uma única solução ter componentes acoplados em uma única estrutura, estes componentes (autenticação, sistema de pagamento, dashboards, etc) são construídos em serviços pequenos e desacoplados, permitindo que estes serviços sejam independentes e escaláveis. Apesar de tudo, estes serviços ainda podem conversar entre sí através de protocolos (HTTP em grande parte).

_Mas como assim independentes e escaláveis? Não seria mais fácil levantar duas instâncias da mesma aplicação?_

De forma alguma quero denegrir o que a arquitetura monolítica trouxe para gente. Pelo contrário, você tem que pensar muito bem sobre qual tipo de arquitetura usar, levando em consideração os requisitos e características que o seu projeto terá. Para fins de demonstração, vamos considerar um grande sistema que tem mais de 20 funcionalidades/módulos/submódulos/etc monolítico, como vamos garantir que:

* _A aplicação continue disponível caso um dos componentes falhem?_ Isso acontece muito quando temos uma aplicação MVC que está muito acoplada e confia demais na renderização no back-end;
* _Poderíamos fazer um deploy sem que haja indisponibilidade total da aplicação?_ Para cada atualização de uma funcionalidade ou módulo, precisamos fazer o deploy da aplicação inteira. Isso demanda **MUITO** tempo, dependendo do tamanho da aplicação;
* _Poderíamos reaproveitar funcionalidades de outras aplicações, sem que haja a necessidade de reescrita de código?_ Estes serviços deverão ser expostos às aplicações que desejam consumir a sua funcionalidade através das chamadas HTTP.
* _Haverá uma menor curva de aprendizado para novos desenvolvedores?_ Um sistema altamente acoplado acaba arruinando as noites de um novo dev, sendo que ele estaria encarregado de apenas um componente.

Então temos o seguinte:

* A gente não precisa punir uma aplicação inteira caso um dos seus serviços esteja com problemas;
* É desnecessário fazer deploy de 20 componentes cada vez que fazemos uma alteração em apenas um componentes. E podemos subir qualquer quantidade de instâncias separadas de quaisquer componentes havendo a necessidade para tal;
* Podemos garantir que uma pessoa não precise reinventar a roda.
* Podemos garantir que essa mesma pessoa se preocupe apenas com a complexidade do próprio módulo que ela esteja desenvolvendo;

_Beleza. Como vamos fazer que os serviços de uma aplicação conversem entre sí? E como poderíamos escalar elas?_

Dai vou precisar explicar sobre um componente extremamente importante...

## Proxy Reverso