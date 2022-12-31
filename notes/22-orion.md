# ORION and the Three Rights: Sizing, Bundling, and Prewarming for Serverless DAGs

[osdi'22](https://www.youtube.com/watch?v=ry4lvOHwPWQ)、[presentation](https://www.youtube.com/watch?v=ry4lvOHwPWQ)、[github](https://github.com/icanforce/Orion-OSDI22)

文章的出发点：观察到 serverless 的执行时延有很大的波动性

- 不同的函数调用可能经过不同的冷启动排队时延
- 不同的函数调用可能有不同的上下文特征
- 不同的函数调用底层使用的基础设施相互异构

因此，使用单一的指标（均值、中位数）无法很好地描述 serverless 函数的执行时延

- 文章进一步提出了 distribution-aware modeling technique 对 serverless 的性能进行建模

文章的主要工作内容：

- 对 serverless 函数的执行时延分布进行了建模刻画，并分析其对资源申请的影响
- 通过 convolution 和 max 两个操作，对串行和并行函数的运行时延分布进行评估
  - 需要考虑不同的函数再同一 stage 和跨 stage 之间的相关性以准确估计联合分布
- 从函数执行时延分布的角度，设计 serverless 常见的优化方法
  - Right-sizing：找到能达到 SLO 的最优资源配置方法
  - Bunding：将一个函数的多个并行实例放在同一个 VM 上执行
  - Right pre-warming：对执行函数的虚拟机进行提前预热

## Motivation

serverless 的运行环境机理：

- serverless 函数在同一个 stage 和不同 stage 之间的执行时延存在相关关系
  - 虽然执行时延的相关性较弱，但对时延估计的误差有明显的影响
  - 相关关系对串行函数和并行函数的执行时延预估具有不同的影响
- serverless 函数需要选择合适的 VM size
  - 在 VM 中不同的资源的扩展大小不一致（CPU、bandwidth、network）
- 选择合适的数量并行函数实例 bundling 到同一个 VM 执行
  - 过少的实例数量存在提升利用率的空间，过多实例数量导致资源竞争

关于 workload latency 的不同刻画角度：

- 不同的 serverless DAG 具有不同的结构，对 E2E latency 有影响
  - 大多数 DAG 深度较小，宽度变化较大
- 不同的 serverless DAG 的调用频率偏度较大，调用频率对冷启动的影响明显
  - 幂律分布：少数几个高调用频率的 DAG 占用相当部分的函数调用比例
  - 随着调用频率的下降，serverless 函数出现冷启动的频率随之增加
  - pre-warming 的必要性：由于幂律分布存在冷启动的函数调用频率不高，使用 keep-alive 是不足以解决该问题
- 不同的 serverless DAG 的执行时延分布不同（violin graph for each DAG）
- 在同一个 serverless DAG 的不同 stages 中，每一个 stage 拥有自己的 latency 分布
- 在 DAG 的一个 parallel stage 中，不同函数执行时延偏度较大
  - 并行阶段中 DAG 的执行时延考虑最慢的一个函数，因此需要考虑函数的联合分布完成 parallel stage 的时延刻画
  - mechanism to mitigate skew of parallel function: 2x for 98.2%
- 在 DAG 的一个函数中，不同函数实例的执行时延偏度较大

文章提出来的三种优化方式，各自从不同的出发点解决 workload latency 的问题：

- Right-Sizing：为 DAG 中不同的函数分配合适的资源，使得 DAG workflow 中所有函数的联合分布满足用户的 SLO
- Bundling Parallel Invocations：将不同的函数绑定到一个 VM 中执行，缓解 parallel stage 的偏度问题
- Prewarming：由于幂律分布的缘故冷启动存在普遍性，因此需要根据 DAG 中函数的执行时延确定 Prewarm 的时机
