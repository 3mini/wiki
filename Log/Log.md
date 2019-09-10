## Log

### Environment

[Filebeat 6.6.2](https://www.docker.elastic.co/#filebeat-6-6-2-)

[Logstash 6.4.0](https://www.docker.elastic.co/#logstash-6-4-0-)

[AWS Elasticsearch version 7.1](https://aws.amazon.com/about-aws/whats-new/2019/08/amazon-elasticsearch-services-announces-support-for-elasticsearch-versions/)

### Architecture

![](images/log_architecture.png)

<details>

<summary> mermaid </summary>

```
graph LR
    subgraph k8s nodes-A
    Application-->|via stdout|Filebeat
    end

    subgraph k8s nodes-B
    Logstash-->aws-es-proxy
    end
   
    subgraph AWS
    Elasticsearch
    end

    Filebeat-->Logstash
    aws-es-proxy-->Elasticsearch
    Logstash-->Slack
```
</details>

### Logstash Error

```
{
  "type"=>"illegal_argument_exception",
  "reason"=>"Limit of total fields [1000] in index [XXXXX-20190909] has been exceeded"
}
```

[index.mapping.total_fields.limit](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping.html) 의 default value 가 1000 인데, 막 쓰고 있다보니 limit 을 넘겼다.

![](images/index_total_fields.png)

이전에는 Kibana - Dev Tools - Console 에서 아래처럼 해결 했었는데,

```sh
PUT XXXXX-*/_settings
{
  "index.mapping.total_fields.limit": 2000
}
```

에러날 때 마다 매번 수정할 수 없을테니, template 에 추가 해두고 당분간 잊어 버리기로.

```sh
PUT _template/MY_TEMPLATE
{
  "order" : 0,
  "index_patterns" : [
    "XXXXX-*"
  ],
  "settings" : {
    "index" : {
      "mapping.total_fields.limit" : 2000,
      "number_of_shards" : "N"
    }
  },
  "mappings" : { },
  "aliases" : { }
}
```
