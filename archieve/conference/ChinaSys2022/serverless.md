# Microservice & Severless

[chinasys 2022 agenda](https://cs.nju.edu.cn/gurong/chinasys_2022fall/agenda-1940.pdf)

## Erms: Efficient Resource Management for Shared Microservices with SLA Guarantees

ASPLOS 2023，Efficient Resource Management System

- 基于优先级进行资源分配

微服务的依赖图：

- 微服务调度服务复杂，深度宽度、多应用共享、串行并行的调度方式
- 资源过度分配以满足端到端的 SLA 要求
- 启发式算法和机器学习算法进行微服务的资源调度

分段线性的延迟预测模型

## No Provisioned Concurrency: Fast RDMA-codesigned Remote Fork for Serverless Computing

将多个机器使用容器等方式，抽象成单个机器向用户提供服务，无需指定到底需要运行多少台机器，OSDI

- start、run、kill，scale on demand
- cold start：docker pull、init container、init runtime
- 冷启动的时间占据大部份的容器运行开销，67%@Edge

业界的解决方案：

- cache (warmstart)：find a cached container、unpause container
  - 需要预置 $O(n)$ 的热容器数量，云供应商无法很好地预估请求到达数量，需要用户自己预估配置热容器的数量
- caching + fork：使用多个机器进行 fork，每台机器只需保持 $O(1)$ 的容器数量
  - checkpoint 保存容器数据、并且需要在分布式文件系统内进行容器的 restore
  - 可以使用 RPC 避免 checkpoint 操作 -> RDMA
  - fork 操作涉及用户数据，只能在同一个应用中进行或者使用操作系统接口进行限制

MITOSIS

fork for serialization-free state transfer:

- 数据传输方式：message passing or cloud storage
- 使用 serialization-free 的方式减少开销
- pre-fetch 和操作系统抽象在 RDMA 中的实现
