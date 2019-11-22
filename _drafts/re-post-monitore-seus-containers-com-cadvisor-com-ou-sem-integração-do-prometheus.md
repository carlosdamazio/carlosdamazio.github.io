---
title: "[Re-post] Monitore seus containers com cAdvisor (com ou sem integração do
  Prometheus)"
project: false
layout: post
date: 2019-11-21 22:00:00 +0000
image: "/assets/images/prometheus_cadvisor_instagram.png"
headerImage: true
tag: []
category: blog
author: carlosdamazio
description: ''

---
Quando o assunto é monitoramento de containers, poucas pessoas costumam confundir a noção do container se algo temporário, por isso não deveríamos nos preocupar com a sua monitoração. Bem, por um lado podemos pensar que não vale a pena por causa do auto-scale das soluções de orquestração, mas grande parte das soluções NÃO precisam de uma solução FULL escalável com orquestração, pois estes não são o _Facebook._ Tomem essa opinião com uma colher de sal, mas toda parte da infraestrutura deveria ser monitorada, inclusive daqueles componentes que são críticos, como as soluções containerizadas.

Em um projeto que participei anteriormente, após o desenvolvimento e implementação da solução, estava muito difícil de monitorar a operação pela falta de integração dos estados dos containers com as soluções de alerta. E nessa procura, eu encontrei o **cAdvisor**!

![](https://github.com/google/cadvisor/raw/master/logo.png)

É uma solução daemon que é executada como container, analisando o uso de recursos do host por parte dos containers. Mas tem uma pegada melhor: ele analisa os processos que ocorrem dentro dos containers do host. Foi criado pela Google e é nativo com o Docker. Se quiser ver o repositório da ferramenta, dá uma olhadinha [aqui](https://github.com/google/cadvisor)!

Podemos usar ele _n_ formas: de forma stand-alone, integrando com o _Elasticsearch_, com o _Kubernetes_, mas estarei mostrando como montar um caso mínimo de forma Stand Alone e/ou com integração com o Prometheus, uma solução de monitoração open-source.

\# Sujando as mãos

As únicas coisas que você vai precisar são o Docker e o Docker Compose, nesse caso. Para o modo stand-alone, não é necessário utilizar o Docker Compose, basta rodar um único comando apenas. No meu caso, eu estou fazendo uso pelo fato de estar integrando com o Prometheus containerizado.

Primeiramente, crie o arquivo de configuração do Prometheus, onde você vai apontar os _targets_ aos nomes dos containers com suas respectivas portas:

{% gist carlosdamazio/ab084ec869e448c8a4b782378b04f291 %}

Com isso fora do caminho, vamos criar o nosso arquivo de docker-compose.yml. Lembrando que devemos setar o nome dos containers porque eles vão ser o FQDN na rede default do Docker. Mapeie o arquivo de configuração do Prometheus no /etc/prometheus/prometheus.yml para sobrescrever o arquivo default do container e também aponte nos _commands_ o parâmetro de arquivo de configuração no diretório do container:

{% gist carlosdamazio/8a2f9d00ce60c94d3af6f8e3eae42530 %}

Agora, para rodar tudo:

    # Docker compose, se você utilizou a integração com o Prometheus
    docker-compose up
    
    # Se você quiser rodar no modo stand-alone
    docker run \\
      --volume=/:/rootfs:ro \\
      --volume=/var/run:/var/run:ro \\
      --volume=/sys:/sys:ro \\
      --volume=/var/lib/docker/:/var/lib/docker:ro \\
      --volume=/dev/disk/:/dev/disk:ro \\
      --publish=8080:8080 \\
      --detach=true \\
      --name=cadvisor \\
      google/cadvisor:latest

## Finalizando

Se quiser verificar se ambos os serviços estão de pé, acessem o \[Localhost\](http://Localhost) na porta 8080 para o cAdvisor ou 9090 para o Prometheus:

[-6bd8ac8c-e8d0-43f5-8263-7133cbafc913untitled](/assets/images/-6bd8ac8c-e8d0-43f5-8263-7133cbafc913untitled "-6bd8ac8c-e8d0-43f5-8263-7133cbafc913untitled")

Prometheus - localhost:9090

cAdvisor - localhost:8080

Você poderá checar a interface da aplicação por conta própria, tente rodar mais containers e verifique as métricas que ela pode fornecer.

Além disso, se você escolheu integrar com o Prometheus, verifique se ele está se conectando com o cAdvisor através do _status → targets_:

!\[\](-d5cd3b20-708f-47d4-ad0c-448f5cdc0ce9untitled)

Se houver sucesso na conexão, você está pronto(a) para fazer queries sobre estatísticas de uso de containers, aumentando o arcabouço de monitoramento das suas aplicações. Quem sabe, poderíamos integrar com ferramentas de alerta...

!\[\](-0e1dfb85-6073-455f-8d96-38d371cf0a3euntitled)

\# Considerações

Esse foi um repost, pois o post original estava no Medium. Eu removi o post original pelo fato de eu não concordar com as práticas que o pessoal utiliza o meio para abusar do mecanismo de Paywall, que incomoda mesmo um usuário pois este não sabe se o link clicado é para um artigo pago ou gratuito. Existem meios melhores e mais justo para arrecadar.

Obrigado por lerem, e se isso te ajudou, divulguem!

    def wat(wat):
    	return None