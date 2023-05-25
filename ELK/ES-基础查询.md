## ES 基础查询
{docsify-updated}

### RestFul API
`curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`
VERB: 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。

```
集群健康:
curl -XGET 'localhost:9200/_cat/health?v&pretty'
GET /_cat/health

节点列表:
curl -XGET 'localhost:9200/_cat/nodes?v&pretty'
GET /_cat/nodes

#### 索引增删查（索引创建好后不允许修改）
列出所有索引:
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
GET /_cat/indices

创建索引，索引名必须小写，不能以下划线开头，不能包含逗号:
curl -XPUT 'localhost:9200/customer?pretty&pretty'
PUT /customer

#### 文档增删改查
将文档放入customer索引，ID置为1:
curl -XPOST 'localhost:9200/customer/1?pretty&pretty' -d'
{
  "name": "John Doe"
}'

POST /customer/1
{
  "name": "John Doe"
}

自动生成ID:
POST /customer
{
  "name": "John Doe2"
}

查询文档：
curl -XGET 'localhost:9200/customer/1?pretty&pretty'
GET /customer/_doc/1 //查询指定ID的文档
HEAD hk_market_code_table/_doc/00555-HKG //只检查对应ID的文档是否存在
POST /customer/_search //查询所有文档,默认返回10条
GET hk_market_code_table/_count 查询某个索引下文档总数


删除索引：
curl -XDELETE 'localhost:9200/customer?pretty&pretty'
DELETE /customer
```