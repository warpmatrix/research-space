# Optimizing Prediction Serving on Low-Latency Serverless Dataflow

[CoRR](https://arxiv.org/abs/2007.05832)

Prediction Serving：为了大型的应用部署完成训练的机器学习模型完成预测任务

- Key Properties：计算密集、低时延要求、由多个阶段复合完成计算
- 现阶段商业提供黑盒框架：将独立的模型部署为单独的微服务，无法提供计算结构、微服务之间关联等信息，需要手动为每一个模型分配容器
- 一些研究提供白盒框架：需要使用特定的 DSL，重写不同的模型

Cloudburst：支持有状态的 FaaS 计算，本文工作的基础平台

- 三种状态共享的方式：函数复合（function composition）、消息传递（message passing）、缓存查找（lookup in cache）
- 使用 Anna 作为 KV 状态数据存储的方式

本文的主要工作：设计了一个面向 Prediction Serving 的 stateful serverless dataflow 系统

- Cloudflow API 数据流模型涉及的三个关键概念：Dataflow、Table、Operators
- Cloudflow 面向的两种控制流：集成（Ensemble）、级联（Cascade）
- 本文设计的系统主要面向交互式的推理任务，通过提供一系列 API 供开发人员构建管道，并且在系统内部进行 prediction serving pipeline 的优化

本文的优化方式主要分为两个方向：

- Dataflow Rewrite：面向 dataflow 拓扑的静态 Cloudflow 编译优化
  - Operator Fusion：尽可能减少避免 data movement
  - Competitive Execution：竞争执行，缓解掉队者的问题，改善 tail latencies
- Dataflow-to-FaaS 的编译优化过程：面向动态运行时的优化决策
  - Operator Autoscaling and Placement：对于瓶颈任务自动扩展投入更多的资源
  - Data Locality：通过将计算部署到离缓存数据较近的位置，减少远程数据传输
  - Batching：对于神经网络任务，使用批处理以更高的时延为代价换取更高的吞吐量、更好的资源利用率、更小的执行成本

实验中使用到的 pipeline：

- Image Cascade：请求的输入可能影响处理复杂度的工作流
- Video Streams：数据密集和计算密集的任务
- Neural Machine Translation：轻量数据超大模型计算密集的工作流
- Recommender System：大规模传输数据的工作流
