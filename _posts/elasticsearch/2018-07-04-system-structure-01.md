---
title: "system structure 01"
date: 2018-07-04 00:06:00
categories: elasticsearch
tag: elasticsearch
---

# Cluster & Node
엘라스틱서치의 가장 큰 시스템 단위 : 클러스터  
하나의 클러스터는 다시 여러 개의 노드로 이루어짐.  

서로 다른 클러스터는 독립적인 시스템

테스트 하려는데 docker elasticsearch image로 하려니 어렵다.  
-> 빠른 테스트를 위해서 docker ubuntu 이미지 위에서 작업해야겠다.

ubuntu version check 방법
> cat /etc/issue
>
> lsb_release -a


```lunux
apt-get update
apt-get install default-jdk
java -version
apt-get install vim
update-alternatives --config java
vim /etc/environment > JAVA_HOME="위명령어 경로"
source /etc/environment
echo $JAVA_HOME
apt-get install wget
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-6.x.list
apt-get update
apt-get install elasticsearch
cd /usr/share/elasticsearch/
apt-get install tree

// 오 이제 되나?
bin/elasticsearch -d
ps -ef | grep elasticsearch
kill 5305
ps -ef | grep elasticsearch

echo 'bin/elasticsearch -d -p es.pid
echo 'kill `cat es.pid`' > stop.sh
chmod 755 start.sh stop.sh

docker run -p 7000-8000:7000-8000

// 안된다...
// 그냥 mac에 설치하자.
```