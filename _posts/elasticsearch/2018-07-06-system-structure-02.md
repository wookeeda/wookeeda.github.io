---
title: "system structure 02"
date: 2018-07-06 00:07:00
categories: elasticsearch
tag: elasticsearch
---

# tar 압축
- 압축하기 : 
tar zcvf test.tar.gz test 
- 압축풀기 : tar zxvf test.tar.gz


# shard & replica
shard
- 루씬에서 사용되는 메커니즘, 데이터 검색을 위해 구분되는 최소의 단위 인스턴스.
- 엘라스틱서치에 색인된 데이터는 여러 개의 샤드로 분할돼 저장됨. 기본 인덱스당 샤드5 복사본5
- 처음 데이터가 색안돼 저장되는 공간을 최소 샤드(primary shard)

replica
- 데이터 손실 방지(fail over), 최초 샤드가 유실되는 경우 복사본을 최초 샤드로 승격시켜 시스템의 무결성 유지
- 성능 향상 목적, 최초 샤드와 복사본을 동시에 검색해서 더 빠르게 데이터를 찾음

ex)
```linux
curl -H 'content-type:application/JSON' -XPUT localhost:9200/magazines -d '
{
  "settings" : {
    "number_of_shards" : 2,
    "number_of_replicas": 0
  }
}'
```
설정파일 : config/elasticsearch.yml
- index.number_of_shards
- index.number_ofreplicas


# network binding & discovery
하나의 서버에서 별도의 설정없이 클러스터명만 같게 하면 노드가 서로 바인딩 됨.  
but 네트워크에 있는 다른 서버의 노드와도 바인딩 필요



# Zen Discovery
멀티캐스트 방식 : 접근할 수 있는 모든 네트워크의 노드를 자동으로 검색하고 같은 클러스터명을 가진 노트끼리 바로 바인딩  
유니캐스트 방식 : 접근할 네트워크 주소를 지정해서 지정한 노드 사이에만 바인딩

```linux
sudo apt-get update
sudo apt-get install -y openjdk-8-jdk

// elasticsearch 설치
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.1.1.tar.gz
tar zxvf elasticsearch-1.1.1.tar.gz

cd elasticsearch

//elasticsearch-cloud-aws 설치
bin/plugin install elasticsearch/elasticsearch-cloud-aws/VERSION

vi ./config/elasticsearch.yml
# yaml 파일은 탭 안됨. 2칸 단위 띄어쓰기
```

```yml
cluster.name: es.test
cloud:
  aws:
    access_key: XXX
    secret_key: XXX
    region: ap-southeast-1
discovery:
  type: ec2
```

> 움.. 이렇게 세팅하고 띄웠는데 안된다..  
> 뭐가 문제일까ㅋㅋ 일단 다음 챕터로 넘어가봅시당

<br>

## aws 공부
- bit.ly/learnawsfree
- bit.ly/aws-day1-todo

## vi 단축키 공부
- https://blog.outsider.ne.kr/540

```vi
/search_text  
  n/N : next, previous
v : visual mode
  y/p : yank(=copy) / paste
b,B / w,W : move each word
0, $ : move front / back
```

## vi tab setting
https://stackoverflow.com/questions/1878974/redefine-tab-as-4-spaces