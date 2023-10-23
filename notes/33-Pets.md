# Pets: A Unified Framework for Parameter-Efficient Transformers Serving

模型微调带来的一系列变种模型 (Parameter-Efficient Transformers, PETs)：这些模型有共享的参数，也存在着各自特定的参数

- 在下游任务中部署多个变种大模型意味着大量的存储开销，GPU 显存很可能无法模型的需求
- 同时，在服务多种模型的情况下 GPU 难以同时存入多种模型，很可能发生换入换出的情况
- 多种不同的模型由于存在权重差异和算法表示差异，难以进行批处理进一步提高了推理开销
- 现有工作缺乏面向 PET 推理任务的作业管理机制和推理引擎
  - 即使存在共享的参数，仍然需要将整个模型拷贝到推理框架中

本文提出了一个面向多任务 PET 的推理框架：Pets

- **统一的表示方式**：实现作业无关的共享算子和特定作业算子的解耦。因此有：
  - $\alpha(T) + \sum_{i=0}^{T-1} \beta_i < \sum_{i=0}^{T-1} \alpha(1)$
- **作业管理机制**：实现 PET 作业的灵活注册和加载
- **作业推理引擎**
  - 提出 PET Coordinated Barching Strategy，实现将不同作业、任意长度的推理请求进行批处理
  - 提出 PET Operator Scheduling Strategy，将算子放置在不同的 CUDA stream 中

论文面向的 4 种 Pet 方法：

- Adapters：$Y_t = \sigma((X_tW+b)W_{down}) W_{up}$
  - $Y_t = X_t W + b$ | $\sigma(\cdot W_{down})W_{up}$
- MaskBert：$Y_t = X_t(W\odot M_t) + b$
  - $Y_t = X_t W + b$ | $-X_t(W\odot \bar{M}_t)$
- Diff-Pruning：$Y_t = X_t (W + \delta_t) + b + b_t$
  - $Y_t = X_t W + b$ | $X_t \delta_t + b_t$
- BitFit：$Y_t =X_t W + b + b_t$
  - $Y_t = X_t W + b$ | $b_t$

基于特定作业算子和作业无关算子的解耦，本次工作进一步提出了后续优化执行的策略：

Coordinated Batch Scheduling：

- 将同一个 task 的不同请求的特定作业算子进行批处理形成 mini-batch
- 将不同 task 的不同请求的作业无关算子进行批处理形成 macro-batch

PET Operator Scheduling：

- 使用 Operational intensity 衡量算子计算强度和访存强度的比率
- 将不同 intensity 的算子映射到不同的 streams，避免对同一资源的竞争
