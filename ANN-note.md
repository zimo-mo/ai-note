# ANN note

## ANN介绍
NNS（Nearest Neighbor Search）作为很多领域的基本构建算法，比如信息检索、模式识别、机器学习、推荐系统等。但随着数据规模的爆炸式增长和维度灾难，精确的NNS搜索无法满足效率和成本的平衡，因此大量基于近似的最近邻搜索（approximate NNS）算法被提出来，主要思想容忍部分精度损失换取搜索时间。
ANNS任务通过特定的索引结构，将整个高维向量空间构建向量索引，映射到低维或者小范围向量空间，在映射后的空间中检索。围绕如何构建索引结构，可将当前ANNS算法分为四大类：hashing-based [40,45];tree-based [8,86]; quantization-based [52,74]; graph-based [38,67]。
