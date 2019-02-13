# ES-Tutorial-1

ElasticSearch 첫번째 튜토리얼을 기술합니다.

본 스크립트는 외부 공인망을 기준으로 작성되었습니다.

## Product 별 버전 상세
```
Product Version. 6.6.0(2019/02/07 기준 Latest Ver.)
```
* [Elasticsearch](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.0.rpm)
* [Kibana](https://artifacts.elastic.co/downloads/kibana/kibana-6.6.0-x86_64.rpm)

최신 버전은 [Elasticsearch 공식 홈페이지](https://www.elastic.co/downloads) 에서 다운로드 가능합니다.

## ElasticSearch Product 설치

이 튜토리얼에서는 rpm 파일을 이용하여 실습합니다.

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo yum -y install git

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ git clone https://github.com/benjamin-btn/ES-Tutorial-1.git

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ cd ES-Tutorial-1

[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1
##################### Menu ##############
 $ ./tuto1 [Command]
#####################%%%%%%##############
         1 : install java & elasticsearch packages
         2 : configure elasticsearch.yml & jvm.options
         3 : start elasticsearch process
         4 : install kibana packages
         5 : configure kibana.yml
         6 : start kibana process
#########################################


[ec2-user@ip-xxx-xxx-xxx-xxx ES-Tutorial-1]$ ./tuto1 1

```

## ELK Tutorial 1 - Elasticsearch, Kibana 세팅

### Elasticsearch
* /etc/elasticsearch/elasticsearch.yml
  - cluster.name, node.name, network.host, http.cors.enabled, http.cors.allow-origin 추가설정
  - **tuto1 2 실행 후 cluster.name 은 unique name 으로 별도 설정 필요**

* /etc/elasticsearch/jvm.options
  - Xms1g, Xmx1g 를 물리 메모리의 절반으로 수정

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo vi /etc/elasticsearch/elasticsearch.yml
+ 
+ ### For ClusterName & Node Name
+ cluster.name: mytuto-es
+ node.name: ip-172-31-14-110
+ 
+ ### For Response by External Request
+ network.host: 0.0.0.0
+ 
+ ### For Head
+ http.cors.enabled: true
+ http.cors.allow-origin: "*"

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo vi /etc/elasticsearch/jvm.options

- -Xms1g
+ -Xms2g
- -Xmx1g
+ -Xmx2g
```

### Kibana
* /etc/kibana/kibana.yml
  - server.host: "0.0.0.0" -> 외부에서 접근 가능하도록 변경
  - elasticsearch.url: "http://localhost:9200" -> 주석해제
  - kibana.index: ".kibana" -> 주석해제

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo vi /etc/kibana/kibana.yml

- #server.host: "localhost"
+ server.host: "0.0.0.0"
- #elasticsearch.url: "http://localhost:9200"
+ elasticsearch.url: "http://localhost:9200"
- #kibana.index: ".kibana"
+ kibana.index: ".kibana"
```

## Smoke Test

### Elasticsearch

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ curl localhost:9200
{
  "name" : "ip-172-31-14-110",
  "cluster_name" : "mytuto-es",
  "cluster_uuid" : "52XfKjycSLCSwXqT_YPMXA",
  "version" : {
    "number" : "6.6.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "a9861f4",
    "build_date" : "2019-01-24T11:27:09.439740Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}

[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ curl -H 'Content-Type: application/json' -XPOST localhost:9200/firstindex/_doc -d '{ "mykey": "myvalue" }'
```

* Web Browser 에 [http://ec2-3-0-99-205.ap-southeast-1.compute.amazonaws.com:9100/index.html?base_uri=http://{FQDN}:9200](http://ec2-3-0-99-205.ap-southeast-1.compute.amazonaws.com:9100/index.html?base_uri=http://{FQDN}:9200) 실행

![Optional Text](image/es-head.png)

### Kibana
* Web Browser 에 [http://{FQDN}:5601](http://{FQDN}:5601) 실행

![Optional Text](image/kibana.png)

## Trouble Shooting

### Elasticsearch
Smoke Test 가 진행되지 않을 때에는 elasticsearch.yml 파일에 기본으로 설정되어있는 로그 디렉토리의 로그를 살펴봅니다.

path.logs: /var/log/elasticsearch 로 설정되어 cluster.name 이 적용된 파일로 만들어 로깅됩니다.

위의 경우에는 /var/log/elasticsearch/mytuto-es.log 에서 확인할 수 있습니다.

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo vi /var/log/elasticsearch/mytuto-es.log
```
