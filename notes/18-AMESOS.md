# AMESoS: a scalable and elastic framework for latency sensitive streaming pipelines

[DEBS'22](https://dl.acm.org/doi/abs/10.1145/3524860.3539642)

AESoS 的主要组件：

- Connector: 数据源头和 Pipeline Manager 之间的连接，从第三方平台订阅事件消息，触发函数流调用。
- Pipeline Management：管理函数流的执行，维护相应的元数据，统计 serverless 执行的运行性能等相关数据。
- Scaler：依据统计数据和预估的请求数量，对容器进行扩展和回收。

文章采用 Holt's 模型对未来的请求进行预估，提前对容器数量进行扩展：

- 根据当前的请求到达时间 $\lambda$ 进行函数的扩容，容器的扩容无法立即完成，因此无法对一个 epoch 时间内的请求进行及时的响应
- 由于未来请求的尖峰以及冷启动的时延，可能导致请求的堆积，造成容器数量的 underestimate
- 使用其他预测算法，如随机森林、weighted KNN，无法很好地捕捉请求数量的变化趋势，导致频繁的请求数量震荡

系统运行的评价指标：

- Pipeline Metrics：deadline、totalEstimate
- PNode Metric：$\lambda$、laxity、avgProcessTime、replicas、$\mu$、queueSize
- Message Metric：timeToDeadline、queuingDelay、processTime、execTime、estimate、laxity

实验比较的 baseline（faas 中函数扩展的方法）：

- Request Per Second (RPS) Scaler: 用户显式地设定每一个函数副本接受的请求数量，根据请求数量动态扩展容器数量
  - 在 OpenFaaS 和 KNative 中设置为默认的扩展方式
- Deadline with Prediction (DwP) Scaler: 监视在函数流运行过程中丢失的请求比例，若有一定比例的请求丢失则对容器进行扩容
- LaSS 中的自动扩展部分
