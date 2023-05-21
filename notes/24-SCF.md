# A Serverless Computing Fabric for Edge & Cloud

[CogMI'22](https://ieeexplore.ieee.org/document/10063372)

- serverless 云原生的后端服务并不是一个简简单单的 toy，还需要引入大量的后台服务 (BaaS)
- 通常 BaaS 服务包括：存储服务、消息中间件、缓存解决方案

serverless 在边缘运行带来的一系列挑战：

- BaaS 长时间运行，在资源受限的边缘场景带来性能压力
- 边缘场景的移动性问题，存储解决方案必须是可调整、可迁移的
- 缓存机制需要得到智能的管理，避免出现边缘节点过载的情况
- 应用部署特别是边缘 AI 方向，边缘设备和模型的异构性给匹配管理带来新的挑战
  - 通过控制平面和数据平面分离，解决部署调度、设备资源动态变化等问题
  - 做到 runtime context- 和 infrastructure-aware
  - 模型分割、专用硬件加速，并且需要设计专门的隔离技术实现这些附加资源的隔离
- 计算隔离问题：资源隔离、**性能**隔离
  - 边缘环境的本性：规模、地理分散
  - 由于地理位置导致可能无法保证计算性能
- FaaS 频繁销毁容器导致的高频请求调度，需要完成调度器同步、冲突解决等问题
- serverless 本身带来的一些问题
  - 冷启动、性能隔离、硬件异构带来的性能差异、对 SLO 的支持有限
  - 数据管理和状态传输、数据位置信息缺失

serverless 的部署优势：

- serverless 的传统优势：弹性可扩展、充分利用边缘资源
- 基于 faas 完成持续交付可以尽可能减少资源消耗
- Serverless Computing Fabric (SCF) 被提出对现有的新兴计算资源进行无缝集成
  - 地理分散、资源异构的计算资源

SCF 的三层架构：

- FaaS Runtime：Functions Controller、Request and data routers
  - Support services access layer、Host capabilities access layer
- FaaS Platform Layer：
  - Support Serverless Functions & Services
  - Core FaaS Platform Runtime Mechanisms
- Infrastructure Management and Orchestration Layer
  - Workload Isolation Manager
  - Cluster & Host Management component

## reference

- At the same time, the Edge-Cloud continuum poses new challenges (e.g., [41])
- Serverless comprising a FaaS platform and a large number of supporting cloud services [21].
- The SCF is a continuation of our previous work in Serverless and Deviceless Edge Computing [16], [26]
- Current approaches [33], [44] explore already the potential of Serverless Edge Computing platforms.
- Lack of performance isolation (a.k.a “noisy neighbor” or performance interference [46])
- Dealing with function failures beyond simple retries [19]
- Concurrency management and transactions [19]
- computational isolation can take many forms. Well known forms include virtualization (VMs and recently Micro VMs) [1], containerization (processes, containers, pods and recently distroless and from scratch containers), microkernels and isolates [3]
- Even multi-level schedulers, such as Mesos [17] and YARN [42], often have a centralized component that might become a bottleneck in the Edge-Cloud continuum
- To realize the scheduler for SCF we will build upon our existing SLO-aware scheduler [31] and redesign it to leverage a distributed, highly scalable architecture
- Stateful serverless approaches [7] have shown that the actor pattern is a suitable abstraction [8].
