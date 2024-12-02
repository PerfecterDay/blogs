
+ `id`: 在一个大的查询语句中每个 SELECT 关键字都对应一个唯一的 id
+ `select_type`: SELECT 关键字对应的那个查询的类型
+ `table`: 表名
+ partitions: 匹配的分区信息
+ type: 针对单表的访问方法
+ possible_keys: 可能用到的索引
+ key: 实际上使用的索引
+ key_len: 实际使用到的索引长度
+ ref: 当使用索引列等值查询时，与索引列进行等值匹配的对象信息
+ rows: 预估的需要读取的记录条数
+ filtered: 某个表经过搜索条件过滤后剩余记录条数的百分比
+ Extra: 一些额外的信息