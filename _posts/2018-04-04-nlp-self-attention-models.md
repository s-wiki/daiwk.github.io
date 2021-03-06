---
layout: post
category: "nlp"
title: "自然语言处理中的自注意力机制（Self-Attention Mechanism）"
tags: [自注意力, self-attention,  ]
---

目录

<!-- TOC -->

- [attention的本质](#attention的本质)
- [multi-head attention](#multi-head-attention)
- [self-attention](#self-attention)

<!-- /TOC -->

attention is all you need的解读可以参考

[https://daiwk.github.io/posts/platform-tensor-to-tensor.html](https://daiwk.github.io/posts/platform-tensor-to-tensor.html)

各种attention model可以参考：

[https://daiwk.github.io/posts/dl-attention-models.html](https://daiwk.github.io/posts/dl-attention-models.html)

本文参考[自然语言处理中的自注意力机制（Self-Attention Mechanism）](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247488035&idx=1&sn=9d0568f58cd85d628fa60ddc33d266e9&chksm=96e9cda3a19e44b5e7ce784d08508ad6d03dcd93c96491dd660af4312b9c67b67457486475ea&mpshare=1&scene=1&srcid=0328RMAtTkf2hZSuXZD5vJBR&pass_ticket=tNNNXIGOajFyoVTQkCkEGcrVM4xaK5lnuItOaXnqkjfkBuTkVoKCva7UoF68PTww#rd)

论文作者之一Lukasz Kaiser的ppt：[https://daiwk.github.io/assets/attention-is-all-you-need-lkaiser.pdf](https://daiwk.github.io/assets/attention-is-all-you-need-lkaiser.pdf)

## attention的本质

Attention函数的本质可以被描述为**一个查询（query）与一系列（键key-值value）对一起映射成一个输出**。分为以下3步：

+ 将**query**和**每个key**进行**相似度**计算得到权重，常用的相似度函数有点积，拼接，感知机等
+ 使用一个**softmax**(因为是一系列的k/v，所以类似多分类，要用softmax)函数对这些**权重进行归一化**
+ 将**权重**和**相应的键值value**进行**加权求和**得到最后的Attention

`\[
attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt {d_k}})V
\]`

目前在**NLP研究**中，key和value常常都是同一个，即 **key=value**(如下例中的源语言的编码器输出)。

对比[https://daiwk.github.io/posts/nlp-nmt.html#4-%E6%B3%A8%E6%84%8F%E5%8A%9B%E6%9C%BA%E5%88%B6](https://daiwk.github.io/posts/nlp-nmt.html#4-%E6%B3%A8%E6%84%8F%E5%8A%9B%E6%9C%BA%E5%88%B6)以及[https://daiwk.github.io/posts/platform-tensor-to-tensor.html#422-attention](https://daiwk.github.io/posts/platform-tensor-to-tensor.html#422-attention)可以发现：

+ 机器翻译里的源语言的编码器输出`\(h_j\)`就是`\(V\)`
+ 机器翻译里的源语言的编码器输出`\(h_j\)`同样是`\(K\)`
+ 机器翻译里的目标语言的隐层状态`\(z_i\)`就是`\(Q\)`
+ 机器翻译里的目标语言和源语言的匹配程度`\(e_{ij}\)`就是`\(\frac{QK^T}{\sqrt {d_k}}\)`
+ 机器翻译里的归一化后的目标语言和源语言的匹配程度`\(a_{ij}\)`就是`\(softmax(\frac{QK^T}{\sqrt {d_k}})\)`
+ 机器翻译里的`\(c_i\)`就是最终的`\(attention(Q,K,V)\)`

## multi-head attention

+ Query，Key，Value最开始都是`\(d_{model}\)`维，各自通过h个线性变换拆成h部分，每一部分的大小是`\(d_k\)`，`\(d_k\)`和`\(d_v\)`（`\(d_k=d_v=d_{model}/h\)`）。
+ 然后这h部分，每部分各自的q/k/v做一个attention的操作得到对应的attention
+ 最后把这h个attention给concat到一起

不同之处在于进行了h次计算而不仅仅算一次，论文中说到这样的好处是可以**允许模型在不同的表示子空间里学习到相关的信息**。


## self-attention

Self-Attention即**K=V=Q**，例如输入一个句子，那么里面的**每个词都要和该句子中的所有词**进行Attention计算。目的是学习句子内部的词依赖关系，捕获句子的内部结构。

使用self-attention的原因：

+ 每一层的复杂度：
    + 如果输入序列n小于表示维度d的话，每一层的时间复杂度Self-Attention是比较有优势的。
    + 当n比较大时，作者也给出了一种解决方案Self-Attention(restricted)即每个词不是和所有词计算Attention，而是只与限制的r个词去计算Attention。
+ 是否可以并行: multi-head Attention和CNN一样不依赖于前一时刻的计算，可以很好的并行，优于 RNN。
+ 长距离依赖: 由于Self-Attention是每个词和所有词都要计算Attention，所以不管他们中间有多长距离，最大的路径长度也都只是 1。可以捕获长距离依赖关系。
