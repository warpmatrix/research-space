# LaSS: Running Latency Sensitive Serverless Computations at the Edge

[github](https://github.com/umassos/lass-serverless)、[HPDC'21](https://dl.acm.org/doi/abs/10.1145/3431379.3460646)：高性能并行和分布式计算（高性能计算顶会）

- principled queuing-based methods：合适的资源分配与自动扩展
- fair-share allocation approach：保证公平的情况下最小资源申请
- resource reclamation methods：资源再分配（over-provision -> under-provision）

serverless 与 edge computing 结合的原因：具有相似的应用场景和需求

- serverless 的设计目标：infrequent or bursty short-lived computations
  - serverless 存在最大执行时间，因此存在执行任务的时延阈值
- edge computing 很多场景下同样注重 latency sensitive, bursty, short-lived request
- edge server 的资源相对受限，因此需要 serverless 范式提高边缘资源的利用率

serverless 与 edge computing 结合的一些挑战：

- edge server 受限的资源和场景波动的请求导致面临资源压力
- serverless 通常设定最大执行时延，可能和 edge server 受限的资源相冲突
  - 需要 edge server 进行合理的资源分配

edge computing 与 serverless 的应用场景：motion-activated smart camera

- 任务执行的 SLO deadline 由 serverless 的最大执行时延和 edge computing 的业务需求决定

文章的主要工作：解决 serverless 与 edge computing 结合所特有的一些问题

- 分析场景中的服务模型：
  - 服务的到达频率在较短时间可以近似为泊松过程
  - 服务模型使用 M/M/c 排队系统平稳随机概率进行描述
  - 资源申请过程估计任务所需的容器资源，逐渐增加函数分配的容器数量，直到满足高可用阈值（99%）
- edge computing 出现资源受限情况下的资源分配问题
  - 在 overload 情况下，按 function 设置的 weight 按比例进行资源分配
  - 保证一定的资源分配同时避免饥饿和不公平情况的发生
- edge computing 出现资源受限情况下的资源回收问题
  - termination：容器停止，进行 CPU 和 MEM 的资源回收
  - **deflation**：资源压缩，对单个函数减少容器数量提高单个容器的资源利用率，从而增加容器的执行的并行度

LaSS 的具体实现：

- LaSS 模块获取容器信息和请求到达速率信息，实现 control path 和 data path 的分离
- service time distribution 的获取：offline profiling、online learning algorithm
- burst 的检测方法：使用长短两个时间窗口检测 request 流量
  - 若短时间窗口和长时间窗口相比，request 流量发生激增（knative）

文章提出方法的一些不足：

- termination：停止后再分配的容器可能产生资源碎片，无法充分利用集群的资源
  - 波动负载导致的 termination 使得 cold-start 时延增加
- 从 no overload 到 overload（新函数的引入），可能导致一些函数分配的容器资源出现较大波动
- 以函数为粒度设置 weight 而不是面向 req 设置权重
  - 无法对一些紧急的 req 进行响应
- 设置公平的算法可能导致 $0.5 + 0.5 < 1 + 0$ 的问题？
  - 但至少保证了系统的可用性，不会因为强行分配资源导致整个系统崩溃
