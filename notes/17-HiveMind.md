# HiveMind: a hardware-software system stack for serverless edge swarms

[ISCA'22](https://dl.acm.org/doi/abs/10.1145/3470496.3527407), [presentation](https://www.youtube.com/watch?v=AQz-8JQXyCQ)

centralized versus distributed exectution:

- centralized: higher performance (compute and memory), resource efficiency
- distributed: low network overheads, without scalability bottlenecks

opportunities of serverless:

- 并行执行：serverless 设计上天然具有的 parallelism
  - 无需考虑占用计算机核数的限制
- 弹性扩展：follows closely the load
- 容错机制：hide the increased workload caused by failed functions
  - 对请求的时延产生退化之前，在新的 core 中重新生成任务

challenges of serverless:

- high latency variablity
  - 在同一个节点上函数之间的相互干扰
  - serverless 框架带来的开销
- serverless 实例化的开销占据处理请求时延中相当大的一部分
  - 特别在边缘场景下的 short-running tasks 实例化开销占比更大
  - 计算密集的任务实例化开销占比相较没有这么大
- 函数之间的通讯开销
  - serverless 的可迁移性导致无法直接和其他的 serverless function 进行通讯
  - serverless 进行通讯的方式：

HiveMind 的组件：

- Domain Specific Language (DSL) and declarative programming interface
- Hybrid Execution Model:
  - 难以评估划分策略的性能和能量消耗，并且边缘场景下负载波动和出错会导致 profile 失效
  - 根据用户需求和性能优化模型，进行任务划分
  - 针对不同的任务划分方式，创建相应的通讯 API
- Scheduler：找到合适的调用器并完成函数之间的信息传输
  - worker monitor：轻量级系统监控进程
  - 两个调度的优化方向：将子函数放置在和父函数相同的容器、避免冷启动减少实例化开销
  - 环境隔离，避免函数之间的干扰：缓存隔离、内存带宽隔离
  - 多个调度器负责系统任务，保证集中式调度器的可拓展性
- Fast Remote Memory Access
  - 通过硬件实现 memory interconnect (RDMA, RoCE)
  - processor's cache coherence protocol 处理脏数据
  - 实现了 serverless 之间的直接通讯，但是仍保持了 serverless 对用户透明可在任意位置运行
- Hardware-based networking acceleration
  - FPGA 作为 NUMA 承载 RPC stack
  - RPCClient/RPCServer：封装 RPC 调用线程池/注册存储远程调用过程
  - hard/soft reconfiguration：不同粒度的配置以达到不同应用的精细优化

HiveMind 的一些特性：

- continueous learning：面向分布式（swarm）弹性（enable/disable）设备的持续学习
- Fault tolerance：通过心跳机制检测设备是否存活，通过负载再分配实现容错
- Straggler mitigation：根据任务的优先级确定落后者的比例，迁移到新的服务器完成执行

硬件缩写：

- RDMA：Remote Direct Memory Access
- EoCE：RDMA over Converged Ethernet
- NUMA：Non-Uniform Memory Access
- NIC：Network Interface Controller
- CCI-P：Core Cache Interface
