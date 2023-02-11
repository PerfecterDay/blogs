# Elasticsearch 简介

我们提过 Elasticsearch 是面向文档的，这意味着索引和搜索数据的最小单位是文档，在 Elasticsearch 中文档有几个重要的属性：
+ 是自我包含的 。 一篇文档同时包含宇段（如 name ）和它们的取值（如 Elasticsearch Denver ）。




### 常用操作 Rest API
0. 创建新索引：`curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X PUT 'localhost:9200/<new_index>'`
1. 查看当前节点的所有 Index: `curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X GET 'https://localhost:9200/_cat/indices?v' `
2. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X  GET 'https://localhost:9200/_search?pretty' 
3. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= 'https://localhost:9200/_mapping?pretty=true'   列出每个 Index 所包含的 Type。


//创建index
curl --location --request PUT 'http://10.187.144.42:8200/market' \
--header 'Content-Type: application/json' \
--data-raw '{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 0,
        "analysis": {
            "analyzer": {
                "my_custom_analyzer": {
                    "type": "custom",
                    "tokenizer": "ngram"
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "instrument": {
                "type": "text",
                "analyzer": "my_custom_analyzer",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 150
                    }
                }
            },
            "market": {
                "type": "text"
            },
            "cname": {
                "type": "text",
                "analyzer": "standard",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 150
                    }
                }
            }
        }
    }
}'


//迁移
curl --location --request POST 'http://10.187.144.42:8200/_reindex' \
--header 'Content-Type: application/json' \
--data-raw '{
    "source": {
        "index": "market_v1"
    },
    "dest": {
        "index": "market"
    }
}'

//删除
curl --location --request DELETE 'http://10.187.144.42:8200/market'
//查询索引数据量
curl http://10.187.144.42:8200/market/_search


curl -X POST -d '{
    "appkey": "D133F72A44D7082C56B1DB3FE0332E33",
    "username": "gta.gtauser2022",
    "password": "gta221p128wd"
   }'  https://gtjuat-data.iqdii.com/jybapp/login/serverlogin
   
   
   curl -X POST -d '{
    "token": "F80B45DA1629746725C2AF4C",
    "username": "test1234",
    "ipaddress":"223.165.23.63"
   }'  https://gtjuat-data.iqdii.com/jybapp/login/getusersession



ElasticsearchAutoConfiguration