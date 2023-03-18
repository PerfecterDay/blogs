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
    "token": "15D44CC971ACB3E5C079DB88",
    "username": "wq9pmpaUblrySna0bE96",
    "ipaddress":"223.165.23.63"
   }'  https://gtjuat-data.iqdii.com/jybapp/login/getusersession


   curl -X POST -d '{
    "token": "15D44CC971ACB3E5C079DB88",
    "username": "wq9pmpaUblrySna0bE96",
	 "packagecode":"30300",
  "month":"1",
   }'  https://gtjuat-data.iqdii.com/jybapp/service/applymarketpkg




curl -X POST -d '{
    "appkey": "D133F72A44D7082C56B1DB3FE0332E33",
    "username": "gta.gtauser2023",
    "password": "gta231p0w1d"
   }'  https://gtadata.iqdii.com/jybapp/login/serverlogin



   curl -X POST -d '{
	"username": "test1234",
    "token": "297F5E0E2964B22BA5FE5011",
    "ipaddress":"223.165.23.63"
   }'  https://gtadata.iqdii.com/jybapp/login/getusersession


ElasticsearchAutoConfiguration




curl -u elastic:password --location --request DELETE 'http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table'
curl -u elastic:password --location --request DELETE 'http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/hk_market_code_table'
curl -u elastic:password --location --request DELETE 'http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/jyb_market_code_table'



curl -u elastic:password --location --request GET 'http://search-service-svc:8000/search/api/indexExists?indexName=hk_market_code_table'

curl -u elastic:password --request PUT 'http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/info' \
--header 'Content-Type: application/json' \
--data-raw '{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 3
  },
  "mappings": {
    "properties": {
      "abst": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "id": {
        "type": "long"
      },
      "pubDt": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "tit": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}'



curl -u elastic:password --location http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/_cat/indices?v

curl -u elastic:password --location http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table/_count
curl -u elastic:password --location http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/hk_market_code_table/_count
curl -u elastic:password --location http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/jyb_market_code_table/_count

curl -u elastic:password --location http://es-sg-vfp331pa70007h94i.elasticsearch.aliyuncs.com:9200/jyb_market_code_table/_search



curl --location 'http://10.4.152.180:8000/user/token/tradeCheck' \
--header 'App-Name: gtjagjapp' \
--header 'App-ID: 3E1EEECDB7A577CCA1A2E2C1885DBE7A' \
--header 'deviceId: 6cba7aed5c92c10d9239b6bba63b61174f4803508af51b99255459646333ad91' \
--header 'short-token: S-a1f7a27c-706d-461c-90a0-7e497e8e5e16' \
--header 'deviceType: 80' \
--header 'platform: IOS' \
--header 'osDevice: Iphone 14 plus' \
--header 'gtja-international-app-token: 5ec3de13-d10a-42c4-9fbb-fc46da029514' \
--header 'Content-Type: application/json' \
--data '{
    "accountCode":"197566"
}'

### 数据导出备份
```
npm install elasticdump -g
elasticdump


elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_data.json \
--type=data

elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_analyzer.json \
--type=analyzer


elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_mapping.json \
--type=mapping
```