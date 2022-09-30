# LIBRA: An Economical Hybrid Approach for Cloud Applications with Strict SLAs

[IC2E 2021](https://ieeexplore.ieee.org/document/9610213)

论文的主要思路：在云计算中心实现 IaaS 和 FaaS 的混用，从而实现在动态变化的需求中。既降低 VM 的冷启动时间（降低程序的响应时间），同时避免突发访问造成 serverless 激增的问题。

- 此前 IaaS 和 FaaS 混用的方式：将部分需求导入到 FaaS，同时对 IaaS 进行扩容减少 SLA 的违反
- 文章提出的混用思路：将低频突发的需求使用 FaaS 进行处理，从而降低应用运行的总成本

文章的主要特点：

- 使用预测器 $avg + \phi \cdot std$ 作为申请 VM 的依据
  - $avg = (1-\alpha) avg + \alpha \times reqs_{cur}$
  - $std = (1-\beta) avg + \beta \times |reqs_{cur} - avg|$, (sample traffic deviatio)
- 并且利用请求数量较少时 Lambda 成本更低、反应更快的特点处理请求
  - 过多的请求被视作请求中短暂的峰值，使用 Lambda 进行处理
  - 较少的请求使用 Lambda 进行处理成本更低

LIBRA 的组成部分：

- scaling manager：接收 traffic monitor 的流量预测信息，调整 IaaS 的扩展策略
- load balancer：监控请求流量，将历史统计结果发送给 traffic monitor
- traffic monitor：接收 load balancer 的统计结果并发送预测流量给 scaling manager

PID 算法进行差错控制：Proportional（比例）、Integral（积分）、Differential（微分）

系统的参数讨论：

- $\alpha, \beta$ 反应 TM 预测流量变化的反应速度
- $K$ 反应 SN 扩展 VM 的反应速度
- $\rho$ 为了保证队列稳定和服务质量，人为设定 VM 服务所占其能力的比例
  - 为了保证请求队列的稳定以及能在在较短的排队时延中请求得到服务，LB 只将一部分请求分配到 VM 进行处理
- $\phi$ 反应预测请求流量后，VM 数量对波动的敏感程度

cite: Spock
