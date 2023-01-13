# percentile pre-calculation

## 一、参考

> [DDSketch: A Fast and Fully-Mergeable Quantile Sketch with Relative-Error Guarantees](https://github.com/Kua-Fu/blog-book-images/blob/main/docs/ddSketch.pdf)

> [Computing Accurate Percentiles with DDSketch](https://www.datadoghq.com/blog/engineering/computing-accurate-percentiles-with-ddsketch/?_gl=1*tc89tr*_ga*NzA2MjcyNzYyLjE2MjgwNjc5NDc.*_ga_KN80RDFSQK*MTY1MTEzMjMxOS4yNTYuMS4xNjUxMTM2NTgxLjA.#how-accurate-are-those-percentiles)

> [Space-Efficient Online Computation of Quantile Summaries](https://github.com/Kua-Fu/blog-book-images/blob/main/docs/quantiles.pdf)

## 二、设计背景

在 `DataFlux`，每天都需要处理大量的分布式数据，如果直接在请求时，实时计算百分位值，会非常消耗资源，甚至请求超时。 需要找到一个好的算法，可以满足下面的几个要求，高效的计算百分位值。

1. 保证计算结果，在一定范围内可靠，即存在相对误差；

2. 计算消耗资源少，即 `cpu/memory` 的消耗在一定可控范围；

3. 可以分布式计算，充分利用集群分布式性能

## 三、为什么百分位计算特殊？

当我们需要计算一些统计数据时候，大多数统计值可以很直观得到结果，例如: 最值(`max/min`)、均值(`avg`)、计数(`count`)，但是百分位无法预聚合。

### 3.1 流式计算

当计算一个不断新增的数据集(`cost_time_list`),的最大值(`max_cost_time`) 时候，我们可以在内存中初始化一个变量(`max_value`)，表示最大值，随着数据的不断新增，该值(`max_value`)也会不断变化；但是，当计算该数据集的百分位值，例如: `P99` ，需要内存中保留所有先前的 `cost_time` 信息，要维护一个排序的列表 `sort_cost_time_list` , 通过计算 `99%` 的 `index` 值，得到 `cost_time` 具体的 `P99` 值。

想象一下，如果数据集有百万/千万，有序列表 `sort_cost_time_list` 将占用大量的内存。


### 3.2 分布式计算

当计算一个分布式数据集的最大值时候，可以先计算出各个独立部分的最大值，然后再次比较这些最大值，可以得到最终的最大值。

![最大值](https://github.com/Kua-Fu/blog-book-images/blob/main/docs/dataflux_percentile/get_max_value.png?raw=true)

但是，我们无法通过简单计算每个子集的百分位值，获取到最终整个数据集的百分位值。

![百分位](https://github.com/Kua-Fu/blog-book-images/blob/main/docs/dataflux_percentile/get_p99_error.png?raw=true)

想要获取到整个数据集的百分位值，需要得到整个数据集的有序列表。这样的方式，我们无法充分利用分布式性能。

![百分位计算](https://github.com/Kua-Fu/blog-book-images/blob/main/docs/dataflux_percentile/get_p99_value.png?raw=true)



