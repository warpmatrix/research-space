# Scissorhands: Exploiting the Persistence of Importance Hypothesis for LLM KV Cache Compression at Test Time

KV Cache 的大小与以下因素成正比：**批处理大小**、**序列长度**、模型维度

文章工作的出发点：获取上下文信息并不需要所有的 token 都存储在内存当中

- 人类可以快速浏览一篇文章而掌握大意

不同 token 的重复注意力模式：段落中部分 token 是被重复关注的

- 关注的 token 和忽略的 token 在不同的 token 中具有相似性
- 判断被关注的标准：`attn_weight > 1/t`
- 只有一小部分的 token 具有较高的注意力分数

Persistence of Importance Hypothesis：Pivotal Tokens

- 前一步具有重大影响的关键 token，才会在未来一步产生重大影响
- 可以用于预测未来生成所用到的重要的 token

推理过程中所使用到的内存布局：model weights、kv cache、activation buffer

文章提出基于历史的 `attn_weight` 选择最不重要的 kvcache 进行丢弃。

- 在保证 kvcache budget 的情况下，每次丢弃 `m` 个 kvcache
- 并且提供了相应的理论分析，证明了丢弃行为带来的误差分析
