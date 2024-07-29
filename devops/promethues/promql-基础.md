# PromQL-基础
{docsify-updated}

> https://prometheus.io/docs/prometheus/latest/querying/basics/

PromQL是Prometheus内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。

在 Prometheus 表达式的表达语言中，一个表达式或子表达式可以计算为以下四种类型之一：
+ `instant vector(瞬时/即时向量)`：一组时间序列，每个时间序列包含一个样本，所有数据样本共享相同的时间戳。
+ `Range vector(范围向量)`：一组时间序列，每个时间序列包含一定范围的时间数据点
+ `Scalar(标量)`：一个简单的数字浮点值
+ `String(字符串)`：一个简单的字符串值。目前未使用

## 时间序列选择器
时间序列选择器负责选择时间序列和原始的或推断的样本时间戳和值。

### 瞬时向量选择器
瞬时向量选择器允许选择一组时间序列和每个时间序列在给定时间戳（时间点）上的单个样本值。   
在最简单的形式中，只需指定一个度量名称，就能得到一个包含有该度量名称的所有时间序列元素的瞬时矢量。

+ `=`: 选择与所提供字符串完全相等的标签
+ `!=`: 选择不等于所提供字符串的标签
+ `=~`: 选择与提供的正则表达式匹配的标签
+ `!~`: 选择与所提供正则表达式不匹配的标签

Regex 匹配是完全锚定的。 env=~"foo "的匹配被视为 env=~"^foo$"。与空标签值匹配的标签匹配器也会选择完全没有特定标签集的所有时间序列。 同一标签名称可以有多个匹配器。
比如有下述时间序列：
```
http_requests_total
http_requests_total{replica="rep-a"}
http_requests_total{replica="rep-b"}
http_requests_total{environment="development"}
```
查询 `http_requests_total{environment=""}` 会匹配到下述序列：
```
http_requests_total
http_requests_total{replica="rep-a"}
http_requests_total{replica="rep-b"}
```

### 范围向量选择器
范围向量选择器的工作原理与瞬时向量选择器类似，只不过它们返回的是当前瞬时的一段范围内的样本。 从语法上讲，在矢量选择符的末尾用方括号（[]）添加一个时间长度，以指定为每个结果范围矢量元素获取多远的时间值。 范围是一个封闭的区间，也就是说，时间戳与范围任一边界重合的样本仍包含在选择范围内。
 
#### 时间长度：
+ `ms` - 毫秒 
+ `s` - 秒 
+ `m` - 分钟 
+ `h` - 小时 
+ `d` - 天, 假定一天总是 24 小时 
+ `w` - 周, 假定一周总是 7d 
+ `y` - 年, 假定一年总是 365d1

这些时间段可以组合使用：
```
5h
1h30m
5m
10s
```

#### Offset 修饰符
而如果我们想查询，5分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢?

Offset 修饰符允许更改查询中各个瞬时和范围向量的时间偏移量。

看下例子：
```
trade_center_req_total{application="trade-center-service",paramCode="1208"}
trade_center_req_total{application="trade-center-service",paramCode="1208"} offset 5m ---> 查询各个瞬时-5分钟前的值，
```
<center><img src="/pics/prometheus-offset.png" alt=""></center>
可以看出 21:05 的数据，在后一个查询中显示到了 21:10 分的数据点位置。相当于在时间轴上往过去的方向平移5分钟显示。

请注意，Offset 修饰符必须紧跟在选择器之后:
```
sum(http_requests_total{method="GET"} offset 5m) // GOOD.
sum(http_requests_total{method="GET"}) offset 5m // INVALID.
```

#### @ 修饰符
使用 @ 修饰符可以更改查询中各个瞬时和范围向量的计算时间。 为 @ 修改器提供的时间是一个 unix 时间戳，并用浮点数描述。请注意， @ 修饰符也必须紧跟在选择器之后:

```
http_requests_total @ 1609746000 //http_requests_total at 2021-01-04T07:40:00+00:00
sum(http_requests_total{method="GET"} @ 1609746000) // GOOD.
sum(http_requests_total{method="GET"}) @ 1609746000 // INVALID.

# offset after @
http_requests_total @ 1609746000 offset 5m  // GOOD.
# offset before @
http_requests_total offset 5m @ 1609746000  // GOOD.
```

## 子查询
子查询允许您针对给定的范围和分辨率运行即时查询。 子查询的结果是一个范围向量。语法如下：
```
<instant_query> '[' <range> ':' [<resolution>] ']' [ @ <float_literal> ] [ offset <duration> ]
```

#### 使用聚合操作
```
# 查询系统所有http请求的总量
sum(http_request_total)

# 按照mode计算主机CPU的平均使用时间
avg(node_cpu) by (mode)

# 按照主机查询各个主机的CPU使用率
sum(sum(irate(node_cpu{mode!='idle'}[5m]))  / sum(irate(node_cpu[5m]))) by (instance)
```

##### 聚合操作
+ sum (求和)
+ min (最小值)
+ max (最大值)
+ avg (平均值)
+ stddev (标准差)
+ stdvar (标准差异)
+ count (计数)
+ count_values (对value进行计数)
+ bottomk (后n条时序)
+ topk (前n条时序)
+ quantile (分布统计)

#### 内置函数
`increase(v range-vector)` 函数是PromQL中提供的众多内置函数之一。其中参数v是一个区间向量，increase函数获取区间向量中的第一个和最后一个样本并返回其增长量：
```
increase(node_cpu[2m]) / 120
```

`rate(v range-vector)` 函数，rate函数可以直接计算区间向量v在时间窗口内平均增长速率。
```
rate(node_cpu[2m])
```


`irate(v range-vector)` irate同样用于计算区间向量的计算率，但是其反应出的是瞬时增长率。irate函数是通过区间向量中最后两个两本数据来计算区间向量的增长速率。这种方式可以避免在时间窗口范围内的“长尾问题”，并且体现出更好的灵敏度，通过irate函数绘制的图标能够更好的反应样本数据的瞬时变化状态。
```
irate(node_cpu[2m])
```