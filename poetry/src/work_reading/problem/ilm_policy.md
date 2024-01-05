# opensearch ilm policy

## 一、参考

> [opensearch ism policy](https://opensearch.org/docs/latest/im-plugin/ism/policies/)

> [opensearch ism apis](https://opensearch.org/docs/latest/im-plugin/ism/api/)

## 二、背景

之前配置索引策略，踩坑，`delete` 阶段的时间参数，采用 `min_index_age`, 例如: 设置保存策略为
