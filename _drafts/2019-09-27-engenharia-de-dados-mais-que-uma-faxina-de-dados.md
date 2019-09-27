---
title: 'Engenharia de Dados: mais que uma faxina de dados'
project: false
layout: post
date: 2019-09-27 03:45:00 +0000
image: "/assets/images/istockphoto-641284068-640x640.jpg"
headerImage: true
tag:
- concepts
- data engineering
category: blog
author: carlosdamazio
description: ''

---
Bem, como eu sempre cometo erros na minha vida, vou cometer mais um em tentar dar nome aos bois na nossa indústria. Eu me tornei no que eu mais temia! Mas calma, não vai ser tão ruim assim, eu prometo. Nós tentamos sempre dar um sentido para as coisas que estão ao nosso redor ou o que elas fazem. Isso é importante e faz parte do sentido da vida (ok, sem viajar tanto), interpretamos o mundo de acordo com a nossa visão e experiência de vida, claro, sempre podendo mudar a nossa interpretação com o passar do tempo de acordo com o aprendizado.

![](https://media.giphy.com/media/l0IypeKl9NJhPFMrK/giphy.gif)

> _Ok, meu filho. Mas aonde diabos você quer chegar?_

Temos certos termos na nossa indústria da TI que não são formais em sua concepção, ao contrário do que a gente tem na Ciência da Computação (compiladores, autômatos finitos, EBNF, camada OSI, etc). Isso geralmente acontece por 2 motivos: ou porque a área é muito recente ou porque a formalização do termo ocorre em assuntos adjacentes.

Nas faculdades, há 5 anos atrás, pelo menos, não se falava em Big Data, nem em Ciência de Dados e muito menos ousavam em _relar a mão_ em Machine Learning. Na Engenharia de Dados, não existe uma definição de _textbook_ formal, só existem conceitos relacionados, fundados em pesquisa e na ciência, que podemos compreender o que este processo abrange.

> ## _O que é Engenharia de Dados, de fato?_

Para entender, a Engenharia de Dados é para a Ciência de Dados assim como a Infraestrutura é para o Desenvolvimento: parceiros. 

> É a área que trata da infraestrutura do armazenamento e manipulação de dados, de quaisquer tipo: estruturados e não-estruturados, e também da modelagem e disponibilização destes dados. 

Você pode achar essa definição muito fraca, mas infelizmente o conceito desta área só fazemos ideia do que seja a partir de conceitos adjacentes, não do que ela é de fato.

Provisionando e sustentando soluções para que possam lidar com diferentes tipos de dados, em diferentes perspectivas e diferentes volumes, podendo chegar a milhões de dados por minuto (aonde eu trabalho, eu posso dizer que lido com 600.000 dados por minuto, cerca de 10.000 dados por segundo de throughput sendo parseados, agregados e consolidados com FOLGA). 

Sem uma base sólida, com dados sólidos e entendíveis, como um profissional de Ciência de Dados ou Machine Learning fará o seu trabalho, sendo o foco principal em DADOS? Se isso não fizer sentido, recomendo a leitura [deste](https://hackernoon.com/the-ai-hierarchy-of-needs-18f111fcc007) artigo da Monica Rogati para que você entenda a utilidade da área nesse contexto e até mesmo em qual contexto ele se insere. Mas talvez você esteja se perguntando...

> ## _Beleza, agora qual é o problema que essa área resolve? Qual é a real utilidade disso?_

![](/assets/images/Figure-2.png)

> _Data lake com dados organizados em zonas. Copyright está na foto._

Muitos dos desafios que enfrentamos na Engenharia de Dados abrangem Streaming Data, Data Lake, Data Warehouse, conceitos que vem de outras áreas como Business Intelligence, Sistemas Distribuídos e Big Data, solucionando problemas que desenvolvedores e cientistas de dados enfrentam quando vão desenvolver soluções e tentam usar seus algorítmos de aprendizagem de máquina, onde um ambiente está repleto de volumetria e variedade enorme com velocidade estupenda.

Tente capturar mais de 1 milhão de dados por [SEGUNDO](https://content.pivotal.io/blog/rabbitmq-hits-one-million-messages-per-second-on-google-compute-engine "pivotal"), como o pessoal da Pivotal fez com [GCP](https://cloud.google.com/ "GCP"), usando um cluster de 30 nós de [RabbitMQ](https://www.rabbitmq.com/ "RabbitMQ"). Apesar de um simples script de ETL no Python ter o seu valor, ele não vai conseguir solucionar o problema nem que use seu mecanismo de pseudoparalelismo ou um host extremamente FODA. Um host pode ter recursos suficientes que o sistema consegue gerenciar, depois disso, é ladeira abaixo. Por isso vem o conceito de Sistemas Distribuídos que conseguem equilibrar a carga de processamento em _clusterização_ para, assim, processar volumes extraordinários como se fosse apenas um único host fazendo isso.

A Engenharia de Dados entrega esses dados que provém de sistemas complexos e gigantes para desenvolvedores e outras áreas que queiram manipular esses dados de forma resumida e limpa para seus devidos fins, sem eles se preocuparem em como adquirir esse dado, adicionando uma camada de abstração. Por isso que temos o conceito de Data Warehouse (esse termo vem do Business Intelligence, lembrem do [Inmon](https://www.amazon.com/Building-Data-Warehouse-W-Inmon/dp/0764599445 "Building Data Warehouse") e [Kimball](https://www.amazon.com.br/Data-Warehouse-Toolkit-Definitive-Dimensional-ebook/dp/B00DRZX6XS/ref=asc_df_B00DRZX6XS/?tag=googleshopp00-20&linkCode=df0&hvadid=379727426149&hvpos=1o1&hvnetw=g&hvrand=8794885456156982741&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1001541&hvtargid=pla-406164365193&psc=1 "Kimball")) e Data Lake, que são abstrações de centralizadores de dados estruturados (que é focado em um Data Warehouse) e não-estruturados (que é focado em um Data Lake).

Enfim, são várias utilidade na área que eu me perco em falar neles. Se nada disso te convencer, sugiro que leia alguns artigos relacionados a Big Data ou Streaming Data. Você vai ficar surpreso com os cases que a galera tenta resolver na ciência.

> ## _Massa, mas existe um certo processo que a Engenharia de Dados tenta se encaixar?_

Sim, e essa base já existia bem antes. Se você compreender o que é o processo de ETL (Extract-Transform-Load), você vai amar isso. Na Engenharia de Dados, temos um conceito de _Data Pipeline_, que é uma esteira de processamento de dados que são provenientes de múltiplas fontes

> ## _Poxa, maravilha! Mas como eu vou saber que isso não vai virar apenas um "fad" e morrer com o tempo?_

![](https://media.giphy.com/media/pPhyAv5t9V8djyRFJH/giphy.gif)

Todas as coisas tem o seu devido valor e resolvem um  problema em específico. Mas se você está preocupado em saber se isso é um _fad_ do momento, você não está entendendo com o que a nossa área realmente se preocupa: é em resolver a porcaria do problema. Quanto mais coisa você souber para resolver problemas, MELHOR! Mas claro, não vá querer se matar em querer aprender tudo (porque você não vai conseguir), aprenda uma coisa e siga em frente.

Você não precisa ser o mais rápido para acompanhar o mercado (lembrando, e vou repetir isso conforme eu for postando por aqui, você NUNCA vai conseguir aprender tudo ou acompanhar tudo), mas nessa maratona, você pode ir mais longe do que alguém que ficou parado.

## Algumas considerações nesse primeiro post

Galera, agradeço por terem lido esse meu primeiro post. Entendam que isso dá um trabalho dos infernos de escrever, até porque exercita uma habilidade minha na qual eu sou horrível, hehe. Mas o meu objetivo com esse blog é repassar conhecimento que possuo, me esforçar em aprender coisas para poder explicar para vocês e passar minha visão de mundo e experiência (mesmo sendo alguém jovem, sempre temos algo para ensinar a alguém). Eu não tenho pretensão clássica como "dinheiro e fama", eu só quero melhorar a minha escrita, aprender coisas e repassar esse conhecimento para vocês. Uma troca justa, nós dois crescemos. Para mim, isso já é o suficiente.

Logo abaixo, na seção de _Links,_ vou deixar alguns recursos para vocês aprenderem mais sobre o assunto que foi explicado no post. Isso vai acontecer em QUASE todos os posts, então fiquem ligados nisso!

Esse post foi mais teórico, mas prometo que nos próximos eu vá colocar algumas coisas em prática. Antes de mais nada, o conhecimento é muito importante e merece ser difundido, então compartilhem esse post com mais pessoas!

Então, inté!

## Links

* [Building the Data Warehouse - Bill Inmon]()
* [The Data Warehouse Toolkit... - Ralph Kimball e cia](https://www.amazon.com/dp/1118530802/ref=pd_lpo_sbs_dp_ss_1?pf_rd_p=7a8f5654-37f5-4688-a266-a74309cad748&pf_rd_s=lpo-top-stripe-1&pf_rd_t=201&pf_rd_i=0764599445&pf_rd_m=ATVPDKIKX0DER&pf_rd_r=QHX66M7VBA9Z2SP5VM4Q&pf_rd_r=QHX66M7VBA9Z2SP5VM4Q&pf_rd_p=7a8f5654-37f5-4688-a266-a74309cad748)
* [Case da Pivotal com o GCP: Cluster de RabbitMQ retém mais de 1 milhão de dados por segundo](https://content.pivotal.io/blog/rabbitmq-hits-one-million-messages-per-second-on-google-compute-engine)
* [The AI Hierarchy of Needs - Monica Rogati](https://hackernoon.com/the-ai-hierarchy-of-needs-18f111fcc007)