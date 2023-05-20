# Elasticsearch 简介
{docsify-updated}

我们提过 Elasticsearch 是面向文档的，这意味着索引和搜索数据的最小单位是文档，在 Elasticsearch 中文档有几个重要的属性：
+ 是自我包含的 。 一篇文档同时包含宇段（如 name ）和它们的取值（如 Elasticsearch Denver ）。

### 常用操作 Rest API
0. 创建新索引：`curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X PUT 'localhost:9200/<new_index>'`
1. 查看当前节点的所有 Index: `curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X GET 'https://localhost:9200/_cat/indices?v' `
2. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X  GET 'https://localhost:9200/_search?pretty' 
3. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= 'https://localhost:9200/_mapping?pretty=true'   列出每个 Index 所包含的 Type。

### Kibana
```
# 创建默认设置的索引
PUT /weather

# 查询所有索引
GET _cat/indices

# 删除索引
DELETE /weather

# 创建索引并设置文档字段
PUT /weather
{
  "mappings": {
    "properties": {
      "user": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_max_word"
      },
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_max_word"
      },
      "desc": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_max_word"
      }
    }
  }
}

#自动生产ID
POST /weather/_doc
{
  "user":"1",
  "title": "2",
  "desc": "test"
}

#指定文档ID
POST /weather/_doc/1
{
  "user":"1",
  "title": "2",
  "desc": "test"
}

# 更新ID=1的文档
POST /weather/_doc/1
{
  "user":"2",
  "title": "2",
  "desc": "test"
}

# 查询文档
GET /weather/_search

# 删除ID=1的文档
DELETE /weather/_doc/1

```

### 数据导出备份
```
npm install elasticdump -g
elasticdump


elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_data.json \
--type=data
--limit=500 //每次导出的数据限制

elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_analyzer.json \
--type=analyzer


elasticdump \
--input=http://elastic:xz3H7rrPCCxrbsdt@es-sg-vfp331pa70007h94i.public.elasticsearch.aliyuncs.com:9200/hk_jyb_market_code_table \
--output=hk_jyb_market_code_table_mapping.json \
--type=mapping
```