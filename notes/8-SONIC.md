# SONIC: Application-aware Data Passing for Chained Serverless Applications

[usenix](https://www.usenix.org/conference/atc21/presentation/mahgoub), [presentation](https://www.youtube.com/watch?v=fAnpvMZvUPI), [Errata Slip](https://www.usenix.org/system/files/atc21_errata_mahgoub.pdf)

论文的主要思路：从用户的角度出发，研究 serverless 中 function 组成的 workflow 其中 function 之间的数据传输方式对性能的影响

- 执行现状：使用远程存储服务，如 AWS S3
- 此前的研究更多是基于远程存储服务进行优化，如：使用内存代替磁盘进行存储、使用不同的介质进行存储
- 优化的目标：满足用户预算目标的基础下，达到最佳的 performance per dollor
- 使用贪心的方法只能得到次优解，因此使用基于迭代的方法（Viterbi algorithm）得到后验最大的解
- 使用 OpenLambda 和 CloudWatch 实现，使用历史数据的加权平均估计数据的传输时间

三种不同的传输方式：新增加的两种传输方式带来的成本可以忽略不计

- VM-Storage：本地虚拟机存储 lambda 运行的结果，要求接下来运行的 lambda 都在同一台虚拟机中执行
  - 节约了网络传输的开销，但引入了 lambda 调度的约束
  - 过多的 lambda 在同一台虚拟机上执行出现内存不足、无法并行处理、热点问题等缺陷，导致计算时延增大
- Direct-Passing：数据存储在本地虚拟机，将文件的访问信息发送给管理器；访问数据的 lambda 通过网络获得本地虚拟机上的数据
  - 和 VM-Storage 相比拥有更高的并行度并且对 lambda 的放置没有约束
  - 但是过多访问数据的 lambda 会导致虚拟机的网络资源成为新的瓶颈
- Remote-Storage：本地虚拟机将数据上传到远程存储系统；访问数据的 lambda 从远程存储系统下载数据
  - 使用专有的存储服务提供了更大的带宽资源，拥有更好的可拓展性，并且访问数据的 lambda 数量的增多也拥有一致的访问数据速度
  - 数据至少需要经过两次复制才能完成传输：source lambda to remote storage 和 remote storage to destination lambda

三种传输方式的影响因素：

- 数据的扇出度：不同方式的可扩展性不同，VM-Storage < Directly Passing < Remote Storage
- 虚拟机的网络带宽资源：Directly Passing 和 Remote Storage 相比，带宽资源越多越倾向于使用 Directly Passing，带宽资源越少越倾向于使用 Remote Storage
- 扇出的方式：Scatter（分散）和 Broadcast（广播）
  - Scatter 传输的数据总量是一个常数
  - Broadcast 传输的数据总量随着扇出度的增加而线性增加
- 数据的局部性和并行程度的权衡：处理大数据的轻量级应用更侧重数据的局部性，处理小规模数据的重应用更侧重并行度
- 串行执行和冷启动之间的权衡：串行执行执行速度慢但可以减少冷启动的开销，并行执行执行速度慢但带来更多的冷启动开销

系统的组成部分：

- 集中管理器：存储文件和服务器的元信息
- 分布式管理器：分布在各个虚拟机中，管理执行数据传输的方式

系统执行的顺序：

0. 系统使用默认的传输方式执行收集训练数据
1. 系统使用收集的数据训练模型得到预测器，预测内存占用、计算时间、中间数据大小等相关参数
2. 通过预测器系统给出数据传输和函数放置的推荐方式

系统实现注意的一些点：

- 数据局部性和并行度的权衡
- 函数串行执行和避免冷启动的权衡
- 考虑冷启动和热启动，权衡是否启用新的虚拟机
- 容错性的实现，要求系统实现函数请求的结果幂等（idempotent）
- 原子可见性（atomic visbility）的实现，系统延迟执行等待同一逻辑请求发送完毕
- 系统对输入内容的敏感程度
- 系统对预测结果误差的容忍性：over-prediction 比 under-prediction 的性能差
- 系统的可拓展性，满足 standard state machine replication (SMR) strategies
- 系统和其它系统的集成：provider-centric metrics 系统、集群计算框架系统 etc.

系统的评估指标：Perf/$ 和 laytency

系统评估的 baseline：OpenLambda + S3、OpenLambda + Pocket、SAND、AWS-$\lambda$、Oracle SONIC

- baseline 设置内存大小的两种方式：memory-sized、latency-optimized

系统测试的三种应用环境：Video Analysis、LightGBM、MapReduce Sort
