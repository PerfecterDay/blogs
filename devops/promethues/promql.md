# PromQL
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


### 简单查询
prometheus_target_interval_length_seconds

单label过滤： 
	prometheus_target_interval_length_seconds{quantile="0.01"}
	prometheus_target_interval_length_seconds{quantile!="0.01"}

多个label过滤：
trade_center_req_err_total{application="trade-center-service", paramCode="1638"}


{__name__=~"trade_center_(req|resp)_err_total"}

#### 瞬时查询
PromQL支持使用=和!=两种完全匹配模式：
+ 通过使用`label=value`可以选择那些标签满足表达式定义的时间序列；
+ 反之使用`label!=value`则可以根据标签匹配排除时间序列；
+ 使用`label=~regx`表示选择那些标签符合正则表达式定义的时间序列；
+ 反之使用`label!~regx`进行排除；

```
http_requests_total{instance="localhost:9090"}
http_requests_total{instance!="localhost:9090"}
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```
直接通过类似于PromQL表达式 `http_requests_total` 查询时间序列时，返回值中只会包含该时间序列中的**最新的一个样本值**，这样的返回结果我们称之为**瞬时向量**。而相应的这样的表达式称之为*瞬时向量表达式*。

#### 范围查询
而如果我们想过去一段时间范围内的样本数据时，我们则需要使用区间向量表达式。区间向量表达式和瞬时向量表达式之间的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器`[]`进行定义
```
http_request_total{}[5m]
```
除了使用m表示分钟以外，PromQL的时间范围选择器支持其它时间单位：
+ s - 秒
+ m - 分钟
+ h - 小时
+ d - 天
+ w - 周
+ y - 年

#### 位移
在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：
```
http_request_total{} # 瞬时向量表达式，选择当前最新的数据
http_request_total{}[5m] # 区间向量表达式，选择以当前时间为基准，5分钟内的数据
```
而如果我们想查询，5分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为offset:
```
http_request_total{} offset 5m
http_request_total{}[1d] offset 1d
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