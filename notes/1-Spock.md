# Spock: Exploiting Serverless Functions for SLO and Cost Aware Resource Procurement in Public Cloud

2019 IEEE International Conference on Cloud Computing, CLOUD

论文的主要思路：

- 使用 VM-based (IaaS) 的方式管理云资源在应对突发访问存在不足：供应过多导致资源的浪费，供应过少导致 SLO 的违反增加
- Serverless (FaaS) 作为一种新出现的云资源使用方式，具有启动时间短、使用时才收费等一系列特点
  - 即使 Serverless 收费标准比 VM 高，但由于 VM 的使用率不能达到 100%，因此使用 Serverless 的总开销比 VM 小同时违反 SLO 的次数也较少
  - 同时，包含其它优势包括用户无需管理自动拓展技术等。

研究背景和动机：

- 主要原因在于 VM 作为分配资源的单元粒度过大：VM 启动时间过长、用户直接租用 VM 即使使用率较低或不使用也需要收费
- 在请求速率固定时，VM 无需考虑扩容问题，成本优于 lambda 同时不会违反 SLO
- 当请求动态变化时，VM 需要维护最大请求数量的资源数，否则会违反 SLO
  - 当平均请求数量较高，和峰值相差不多使用 VM 成本更低
  - 当平均请求数量较少，峰值和平均值相比较高使用 lambda 成本更低
- 因此，在峰值的平均值的某一比值之间可以进行 VM 与 lambda 的 tradeoff

VM 和 lambda 的一些使用细节：

- Lambda 也需要用到存储系统 (remote store)，如：AWS S3 Bucket；VM 的模型从 EBS 中得到
- VM 中一个 vCPU 可以处理多个请求，即一个 VM 可以处理多个请求
- lambda 计算的总成本：$C = N \times (a \times M \times E + b)$
  - $M$：lambda 使用的内存大小；$E$：lambda 函数的执行时间；$N$：lambda 函数的执行次数

此前的研究：效果不佳的根本原因为 VM 的申请粒度过大无法优化负载激增的情况，需要支付更高的成本应付

- 研究 VM 的自动伸缩策略和采购类型
- 预测请求的峰值负载主动进行伸缩配置
- 可以对容器进行微调使其保持热启动进一步减少

## Spock

系统的运行框架：查看 VM 是否存在 free slots 可以进行分配；若不存在请求运行新的 lambda function，并进行相应的扩展策略。

主要的组件：

- Resource Manager：请求的编排和资源的请求与释放
- Load Balancer：依据预测的负载确定资源的请求数量
- Scaling Policy：根据定义的 scaling policy，预测负载和资源的激活情况
- Load Monitor：监控 VM 和 lambda 资源的使用情况

两种扩展策略：

- Reactive：根据请求速率判断是否需要扩展 VM 的数量
- Predictive：使用移动窗口线性回归模型预测未来一段时间的请求数量
  - 方便提前进行虚拟机的申请：使用最终时刻申请 VM？使用窗口内均值申请虚拟机？

运行和比较的细节：

- 资源的释放：VM 经过三分钟的空闲时间会被释放
- 运行打包：请求到来时优先使用 VM，尽可能释放资源降低成本
- 每个请求（机器学习模型）可能申请不同的内存大小
- 若只使用 VM 由于启动延时，VM 需要处理在队列中积压的请求，请求数量减少时 VM 的数量延迟一段时间才会减少
- 若使用 VM 和 lambda，请求数量上升时使用 lambda 进行处理，VM 的数量和请求数量的变化基本保持一致
- 更频繁的请求峰值可能导致 spock 的成本上升（需要更频繁地使用 lambda），请求峰值更高 spock 的成本会下降无需维护过多的 VM

some interchangeably conception:

- Lambda, serverless
- free slots, free vCPU

Q&A:

- Figure2 中 resource demand (in blue) 没有超过 VMs (in green) 为什么还会出现 SLA (in red)
- spot VM instances?
