# Costless Performance Optimization (Appendix)

abbreviation & conception:

- SLO: Service Level Objective
- SLA: Service Level Agreement
- SLI: Service Level Indicator
- S3: Simple Storage Service
- bucket: 云上存储数据的管理单元（计费、权限控制、声明周期）
- EBS: Elastic Block Store

做调度类算法有不同的角度：云服务提供商、客户本身的性能和成本优化

云计算涉及的主要服务：

- VM: pay by leasing time no matter how the utilization ratio
- Serverless (lambda): pay by func calc time and # invoked
- Greengrass: pay by # device

serverless 和 edge server 发展的一些脉络：

0. 使用 VM 比较粗粒度地使用云服务提供商提供的资源
1. 讨论云服务提供商引入 serverless 为 VM 的启动和成本提供优化
2. serverless 组成的 pipeline 在云边进行协同合作，为 FaaS 的整体优化提供了相应的方向
3. 利用不同 edge server / end device 在时间和空间上的关联协同完成任务
   - 存在时空重叠之间的协同
   - 不同时空之间的协同
4. 基于边缘设备的横向工作量负载平衡
5. 基于函数调度而不是流水线的配置调整策略

serverless 视频分析中的一些细节：

- serverless 组成的 pipeline 具有可以优化的空间
- 视频分析中需要进行大数据的视频传输为 serverless 的使用提供了挑战
- 视频分析中的数据往往具有大量的冗余信息：
  - 同一个摄像头视频内容的时空关联、不同摄像头之间的时空关联
- 视频分析中的输入可能对 pipeline 运行的数据大小有影响
- 视频分析中可能存在分支或其他复杂结构并行运行的 pipeline

serverless 强调低时延的场景：

- 请求量大的场景：需要在限定的时间内完成所有请求的计算，否则将会出现请求的丢失
  - 视频分析、大规模物联网场景
- 快速决策的场景：无人机飞行、无人驾驶
- 屏蔽了具体的执行环境细节，可以在 cloud 和 edge 之中进行迁移（效果等同于跨云协同）

边缘场景的计算环境：边缘只是一个相较而言的概念，面向不同的层级关系，边缘场景拥有不同的计算资源

- 局域网的计算资源（边缘盒子）、城域网的计算资源（小型基站）、广域网的资源资源（边缘数据中心）

k8s 视角下的容器技术架构：

- 客户端（cri - crictl, kubelet）：容器编排
- 容器引擎（docker, containerd）：与应用相关的容器管理，容器生命周期、命名空间管理
- 底层容器运行时（runc、kata、wasmedge）

和业界后续讨论的一些问题：

- 边缘坏境下的冷启动解决方案 (ipads 在云上的解决方案)
- 业界边缘 faas 是否会遇到资源限制的问题、解决方案、拉取的方式
- faas workload 的预测问题（特别是微服务转向 faas 的场景）

wasmedge 在 docker 中的实现 from [wasmedge doc](https://wasmedge.org/docs/develop/deploy/using-wasmedge-in-docker/)：

> However, this approach still requires starting up a Linux container. The containerized Linux OS, however slim, still takes 80% of the total image size. There is still a lot of room for optimization. The performance and security of this approach would not be as great as running WebAssembly applications directly in crun or in a containerd shim.

后续投稿可以考虑的方向：

- mobisys poster
- mobisys workshop
- DTMS 2023
