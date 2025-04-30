# 数据类型
{docsify-updated}

## 字符串
字符串类型是 Redis 最基础的类型，所有的键都是字符串类型。字符串的值实际可以是字符串、数字，甚至二进制（图片、音视频），但是大小最大不能超过512MB。
<center><img src="pics/redis-string.png" width="50%"></center>

1. 命令
   + `set key value [ex seconds] [px milliseconds] [nx|xx]` : 设置值，ex 代表设置秒级过期时间，px代表设置毫秒级过期时间，nx代表键不存在才可以设置，xx代表存在才可以设置（用于更新）
   + `setex` : 相当于上面的 ex 选项
   + `setnx` ：相当于上面的 nx 选项，不存在时才设置成功。
   + `mset key value [key value...]` : 批量设置值
   + `get key` ：获取 key 对应的值，key 不存在则返回 nil。
   + `mget key [key...]` : 批量获取值
   + `incr key` ：计数自增,若值不是整数，返回错误；若值是整数，返回自增后的结果；键不存在，按照值为0自增，返回结果1。
   + `decr key` ：计数递减1
   + `incrby key increment` ：计数 + increment
   + `decrby key decrement` ：计数 - decrement
   + `incrbyfloat key increment` ：浮点数 + increment
   + `append key value` ：向字符串尾部追加值
   + `strlen value` ：获取字符串长度
   + `getset key value` ：设置并返回原值
   + `setrange key offeset value` ：设置指定位置的字符
   + `getrange key start end` ：获取部分字符串
  <center>
  <img src="pics/redis-string-time.png" width="60%">
  </center>
   
2. 内部编码
   字符串类型的内部编码有3种：
   + int：8个字节的长整型。
   + embstr：小于等于39个字节的字符串。
   + raw：大于39个字节的字符串。

## 哈希
  <center>
  <img src="pics/redis-hash.png" width="40%">
  </center>


1. 命令
   + 设置值 ：`hset key field value`,设置 field-value，成功返回1，否则返回0。还有 `hsetnx` 命令，只有field不存在时才会设置值
   + 获取值 ：`hget key field`；如果 field 不存在，返回 nil
   + 删除 field ：`hdel key field [field ...]`：可以删除一个或多个 field，返回成功删除的个数
   + 计算field的个数 ：`hlen key`
   + 批量设置field-value ：`hmset key field value [field value ...]`
   + 批量获取field-value ：`hmget key field [field....]`
   + 判断 field 是否存在 ：`hexists key field`,存在返回1，否则返回0
   + 获取所有 field ：`hkeys key`
   + 获取所有 value ：`hvals key`
   + 获取所有 field-value ：`hgetall key`
   + 指定 field 加1：`hincrby key field`
   + 同上，对浮点数操作 ：`hincrbyfloat key field`
   + 计算value的字符串长度 ：`hstrlen key field`

<center>
<img src="pics/redis-hash-time.png" width="50%">
</center>
   

## 列表
## 集合
## 有序集合
