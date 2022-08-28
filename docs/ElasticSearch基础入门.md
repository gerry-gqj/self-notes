# ElasticSearch基础入门

安装

```shell
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/docker.html
```



```http
## 创建索引
PUT http://192.168.1.6:9200/shopping
```

```http
## 删除索引
DELETE http://192.168.1.6:9200/shopping
```





```http
GET http://192.168.1.6:9200/_cat/indices?v
```





