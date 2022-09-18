# ELK


## ElasticSearch
下载解压后，直接运行 `./bin/elasticsearch` 即可运行。
如果这时报错"max virtual memory areas vm.maxmapcount [65530] is too low"，要运行下面的命令。
```
sudo sysctl -w vm.max_map_count=262144
```
默认情况下，Elastic 只允许本机访问，如果需要远程访问，可以修改 Elastic 安装目录的config/elasticsearch.yml文件，去掉network.host的注释，将它的值改成0.0.0.0，然后重新启动 Elastic。
```network.host: 0.0.0.0```
上面代码中，设成0.0.0.0让任何人都可以访问。线上服务不要这样设置，要设成具体的 IP。


Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  dDqETfTPxoSu0aCkRJCS

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  5e0c32a0f3548b39e0d1d532c8673c8c2d6a8f14fdea208f539d320dc31fd56c

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjQuMSIsImFkciI6WyIxMC4xNzYuODEuMjM6OTIwMCJdLCJmZ3IiOiI1ZTBjMzJhMGYzNTQ4YjM5ZTBkMWQ1MzJjODY3M2M4YzJkNmE4ZjE0ZmRlYTIwOGY1MzlkMzIwZGMzMWZkNTZjIiwia2V5IjoiaExJa1A0TUJNUHRlbDNGZVpYaHc6bmw4ZWV4NUZRbE9QT2EzYmNWYk9JdyJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.

### 基本概念
1. 索引-Index
	Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。
2. 文档-Document
	Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。
3. 类型-Type
	Document 可以分组，比如weather这个 Index 里面，可以按城市分组（北京和上海），也可以按气候分组（晴天和雨天）。这种分组就叫做 Type，它是虚拟的逻辑分组，用来过滤 Document。  
	不同的 Type 应该有相似的结构（schema），举例来说，id字段不能在这个组是字符串，在另一个组是数值。这是与关系型数据库的表的一个区别。性质完全不同的数据（比如products和logs）应该存成两个 Index，而不是一个 Index 里面的两个 Type（虽然可以做到）。


1. curl -k -X  GET 'https://localhost:9200/_cat/indices?v' 查看当前节点的所有 Index。
2. curl -k -X  GET 'https://localhost:9200/_search?pretty' 
3. curl -k 'https://localhost:9200/_mapping?pretty=true'   列出每个 Index 所包含的 Type。


elasticsearch -d -p pid
elasticsearch-create-enrollment-token --scope kibana
https://10.176.81.23:9200/

logstash -f config/logstash.conf
  bin/kibana


## Kibana
home -> Manage index lifecycles -> Data Views -> create data view
