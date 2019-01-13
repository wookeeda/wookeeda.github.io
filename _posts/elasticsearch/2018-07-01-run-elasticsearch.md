---
title: "run elasticsearch"
date: 2018-07-01 20:15:00
categories: elasticsearch
tag: elasticsearch, docker
---

1. docker에서 설치  
참고) https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

    ```linux
    $docker run -it -p 80:80 -p 9200:9200 ubuntu:16.04

    #apt-get update
    #apt-get install -y default-jdk
    #update-alternatives --config java
    #apt-get install -y apt-utils vim wget curl apt-transport-https
    #wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
    #echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-6.x.list
    #apt-get update
    #apt-get install -y elasticsearch
    #ctrl + p + q

    $docker ps -a
    $docker commit container_id ub_es:1.0
    $docker run -it -p 80:80 -p 9200:9200 ub_es:1.0

    #cd /usr/share/elasticsearch
    #echo 'bin/elasticsearch -d -p es.pid' > start.sh
    #echo 'kill `cat es.pid`' > stop.sh
    #chmod 755 start.sh stop.sh
    ```

2. dockerfile  
    움.. 왜 안되는걸까..  

    ```dockerfile
    FROM ubuntu:16.04

    RUN apt-get update
    RUN apt-get install -y apt-utils vim default-jdk wget curl apt-transport-https
    RUN wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
    RUN echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-6.x.list
    RUN apt-get update
    RUN apt-get install -y elasticsearch

    WORKDIR /usr/share/elasticsearch/

    RUN echo 'bin/elasticsearch -d -p es.pid' > start.sh
    RUN echo 'kill `cat es.pid`' > stop.sh
    RUN chmod 755 start.sh stop.sh

    EXPOSE 80
    EXPOSE 9200

    CMD /bin/bash
    ```
    
    ```linux
    docker build -t ub_es:1.0 ./
    ```



3. dockerfile 안되네.  
    docker image로 다시 실행  
    결국, docker image로 실행  
    ```linux
    docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.0
    ```