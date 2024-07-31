# PromQL 函数
{docsify-updated}

rate(v range-vector) calculates the per-second average rate of increase of the time series in the range vecto∏worr. Breaks in monotonicity (such as counter resets due to target restarts) are automatically adjusted for. Also, the calculation extrapolates to the ends of the time range, allowing for missed scrapes or imperfect alignment of scrape cycles with the range's time period.

irate(v range-vector) calculates the per-second instant rate of increase of the time series in the range vector. This is based on the last two data points. Breaks in monotonicity (such as counter resets due to target restarts) are automatically adjusted for.

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