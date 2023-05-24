## ES 基础查询


### 
集群健康:
curl -XGET 'localhost:9200/_cat/health?v&pretty'
GET /_cat/health

节点列表:
curl -XGET 'localhost:9200/_cat/nodes?v&pretty'
GET /_cat/nodes

### 索引增删查（索引创建好后不允许修改）
列出所有索引:
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
GET /_cat/indices

创建索引:
curl -XPUT 'localhost:9200/customer?pretty&pretty'
PUT /customer

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
POST /customer/_search //查询所有文档

删除索引
curl -XDELETE 'localhost:9200/customer?pretty&pretty'
DELETE /customer