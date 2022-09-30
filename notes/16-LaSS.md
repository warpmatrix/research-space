# LaSS: Running Latency Sensitive Serverless Computations at the Edge

[HPDC'21](https://dl.acm.org/doi/abs/10.1145/3431379.3460646)：高性能并行和分布式计算（高性能计算顶会）

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

serverless 与 edge computing 结合的模型建立：

- 服务的到达频率在较短时间可以近似为泊松过程，相关的服务模型可以使用排队系统进行描述
- M/M/c 排队系统平稳随机概率
