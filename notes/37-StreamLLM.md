# Efficient Streaming Language Models with Attention Sinks

对于 LLM 来说，要推广到比它们进行预训练的更长的序列长度是非常具有挑战性的

- 例如 Llama-2 的最大序列长度为 4K
- 造成这一问题的本质原因是 LLM 在预训练的过程中受到有限上下文窗口的约束，和具体部署的情景存在 gap

该工作的核心目标：deploy an LLM for infinite-length inputs without sacrificing efficiency and performance

## Observation

在进行无限长序列推理的过程中，主要面临以下挑战：

- decode 阶段缓存的 KV Cache 造成了内存开销和解码时延增加
- 当序列长度超过预训练期间设置的注意力机制的上下文窗口大小时，性能会下降

目前的一些解决方法：可接受的序列长度仍然本质上是有限的，不允许持续部署

- sliding window attention
- sliding window attention with re-computing

大量的注意分数被分配给初始令牌，与它们是否与语言建模任务相关无关，如图2所示。我们将这些令牌称为“关注汇”。

- 在前几层的注意力上下文窗口表现出“局部”模式，即最近的令牌受到更多的关注
- 在更深层的注意力上下文窗口，模型的所有层和注意力头都强烈关注初始令牌

随着序列长度的增加，如果将关注汇丢弃将会导致 LLM 的性能发生剧烈变化

## Solution

作者给出的解释是：随着模型层数的增加，hidden_state 已经足以用于表示序列的语义信息，因此需要剔除多余的注意力权重，将多出来的注意力权重置于关注汇（后文可见、信息量较少）

因此本次工作引入 attention sink token 用于推理和训练

在推理时保留前面几个 attention sink token，后面的 attention 使用 sliding window attention

- 这样可以避免重新训练，并且可以将 ppl 维持在一个较低的水平

如果能重新训练，作者引入 trainable attention sink 同样可以获得较好的效果

## Reference

> Despite substantial efforts to expand this window size (Chen et al., 2023; kaiokendev, 2023; Peng et al., 2023) and improve training (Dao et al., 2022; Dao, 2023) and inference (Pope et al., 2022; Xiao et al., 2023; Anagnostidis et al., 2023; Zhang et al., 2023b) efficiency for lengthy inputs, the acceptable sequence length remains intrinsically finite, which doesn’t allow persistent deployments.
>
