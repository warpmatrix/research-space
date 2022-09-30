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

排队论模型的一些补充：

- M/M/C 模型：泊松分布、负指数分布、共有 $c$ 个服务窗口的排队系统

开题报告需要询问的一些问题：

- serverless 实现的方式：云上实现、边缘服务器实现
- edge server 的从属关系：用户、云服务提供商、路由器？
- 一些概念：cloud native, serverless, edge server
