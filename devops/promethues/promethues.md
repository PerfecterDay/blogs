# Promethues
{docsify-updated}

> https://www.prometheus.wang/promql/prometheus-query-language.html

- [Promethues](#promethues)
	- [时序数据概述](#时序数据概述)
	- [Promethues监控指标](#promethues监控指标)
		- [指标(Metric)](#指标metric)
			- [Metric类型](#metric类型)
	- [PromQL](#promql)
		- [瞬时查询](#瞬时查询)
		- [范围查询](#范围查询)
		- [位移](#位移)
		- [使用聚合操作](#使用聚合操作)
			- [聚合操作](#聚合操作)
		- [内置函数](#内置函数)

### 时序数据概述
Prometheus读写的是时序数据，与一般的数据对象相比，时序数据有其特殊性，tsdb对此进行了大量针对性的设计与优化。因此理解时序数据是理解Prometheus存储模型的第一步。通常，它由如下所示的标识和采样数据两部组成：

`标识 -> {(t0, v0), (t1, v1), (t2, v2), (t3, v3)...}`

标识用于区分各个不同的监控指标，在Prometheus中通常用指标名+一系列的label唯一地标识一个时间序列。如下为Prometheus抓取的一条时间序列，其中http_request_total为指标名，表示HTTP请求的总数，它有path和method两个label，用于表示各种请求的路径和方法。

`http_request_total{path="/", method="GET"} -> {(t0, v1), (t1, v1)...}`

事实上指标名最后也是作为一个特殊的label被存储的，它的key为__name__，如下所示。最终Prometheus存储在数据库中的时间序列标识就是一堆label。我们将这堆label称为series。

`{__name__="http_request_total", path="/", method="GET"}`

采样数据则由诸多的采样点（Prometheus中称为sample）构成，`t0, t1, t2...`表示样本采集的时间，`v0, v1, v2...`则表示指标在采集时刻的值。采样时间一般是单调递增的并且相邻sample的时间间隔往往相同，Prometheus中默认为15s。而且一般相邻sample的指标值v并不会相差太多。基于采样数据的上述特性，对它进行高效地压缩存储是完全可能的。Prometheus对于采样数据压缩算法的实现，参考了Facebook的时序数据库Gorilla中的做法，通过该算法，16字节的sample平均只需要1.37个字节的存储空间。

### Promethues监控指标
Prometheus可以采集到当前主机所有监控指标的样本数据:
```
# HELP node_cpu Seconds the cpus spent in each mode.
# TYPE node_cpu counter
node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 3.0703125
```
node_cpu和node_load1表明了当前指标的名称、大括号中的标签则反映了当前样本的一些特征和维度、浮点数则是该监控样本的具体值。

Prometheus会将所有采集到的样本数据以时间序列（time-series）的方式保存在内存数据库中，并且定时保存到硬盘上。time-series是按照时间戳和值的序列顺序存放的，我们称之为向量(vector). 每条time-series通过指标名称(metrics name)和一组标签集(labelset)命名。

在time-series中的每一个点称为一个样本（sample），样本由以下三部分组成：

+ 指标(metric)：metric name和描述当前样本特征的labelsets;
+ 时间戳(timestamp)：一个精确到毫秒的时间戳;
+ 样本值(value)： 一个float64的浮点型数据表示当前样本的值。

```
http_request_total{status="200", method="GET"}@1434417560938 => 94355
http_request_total{status="200", method="GET"}@1434417561287 => 94334

http_request_total{status="404", method="GET"}@1434417560938 => 38473
http_request_total{status="404", method="GET"}@1434417561287 => 38544

http_request_total{status="200", method="POST"}@1434417560938 => 4748
http_request_total{status="200", method="POST"}@1434417561287 => 4785
```

#### 指标(Metric)
在形式上，所有的指标(Metric)都通过如下格式标示：
```
<metric name>{<label name>=<label value>, ...}
```

##### Metric类型
在Prometheus的存储实现上所有的监控样本都是以time-series的形式保存在Prometheus内存的TSDB（时序数据库）中，而time-series所对应的监控指标(metric)也是通过labelset进行唯一命名的。
Prometheus定义了4中不同的指标类型(metric type)：
+ Counter（计数器）: 只增不减的计数器,一般在定义Counter类型指标的名称时推荐使用_total作为后缀。
+ Gauge（仪表盘）: 可增可减的仪表盘,Gauge类型的指标侧重于反应系统的当前状态,因此这类指标的样本数据可增可减。
+ Histogram（直方图）:Histogram和Summary主用用于统计和分析样本的分布情况。
+ Summary（摘要）:Histogram和Summary主用用于统计和分析样本的分布情况。

### PromQL
PromQL是Prometheus内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。

在 Prometheus 表达式的表达语言中，一个表达式或子表达式可以计算为以下四种类型之一：
+ `instant vector(瞬时/即时向量)`：一组时间序列，每个时间序列包含一个样本，所有数据样本共享相同的时间戳。
+ `Range vector(范围向量)`：一组时间序列，其中包含每个时间序列随时间变化的一系列数据点
+ `Scalar(标量)`：一个简单的数字浮点值
+ `String(字符串)`：一个简单的字符串值。目前未使用

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