# Elasticsearch 简介

我们提过 Elasticsearch 是面向文档的，这意味着索引和搜索数据的最小单位是文档，在 Elasticsearch 中文档有几个重要的属性：
+ 是自我包含的 。 一篇文档同时包含宇段（如 name ）和它们的取值（如 Elasticsearch Denver ）。




### 常用操作 Rest API
0. 创建新索引：`curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X PUT 'localhost:9200/<new_index>'`
1. 查看当前节点的所有 Index: `curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X GET 'https://localhost:9200/_cat/indices?v' `
2. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= -X  GET 'https://localhost:9200/_search?pretty' 
3. curl -k -u elastic:+wh_NUsb-BSJQPFHDQ5= 'https://localhost:9200/_mapping?pretty=true'   列出每个 Index 所包含的 Type。