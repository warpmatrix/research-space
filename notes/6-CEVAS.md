# Towards Cloud-Edge Collaborative Online Video Analytics with Fine-Grained Serverless Pipelines

ACM Multimedia Systems Conference (MMSys) 2021

CEVAS: Cloud-Edge collaborative Video Analytics system empowered by fine-grained Serverless pipelines

- serverless 在视频领域的具体结合
- 根据视频内容动态调整任务划分节点
- 视频内容随着时间负载动态变化

领域最主要的问题：

- 数据冗余稀疏，存在兴趣事件的概念（真正有价值的事件发生的时段较短）
- 细粒度的服务编排，形成服务流水线
- 多租户的服务方式，造成请求数据动态波动
- 模型的不同划分位置，云边形成协同合作
- 其它问题：设备异构性、数据异构性

视频分析的两个度量指标：

- average objects per frame, AOPF：每帧平均目标数量
- average unique objects per frame, AUOPF：每帧平均唯一目标数量

系统设计架构：User, Controller, Edge server and Cloud

- 不同的流查询流水线 (stream query pipeline)：不同的设备和不同的流查询流水线
- Controller 的三个组件：Monitor, Predictor, Scheduler

文章主要建模的任务：如何对请求进行划分实现云端的协同运行（多维度多选择背包问题）

系统评价指标：吞吐量、云开销、传输数据、成本

不同的 baseline：

- PureCloud、PureEdge
- SVESC: Slim version of VideoEdge with Serverless Computing supports
