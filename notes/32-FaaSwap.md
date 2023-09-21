# FaaSwap: SLO-Aware, GPU-Efficient Serverless Inference via Model Swapping

目前机器学习任务推理的现状：

- 目前服务器无感知计算对 GPU 计算任务缺乏足够的支持
- 模型执行时间对比：冷启动 10s 执行时间，热启动 20ms 执行时间，两个数量级的时延差距
  - 冷启动包含以下的不同部分：GPU 容器的设置、ML 框架的准备、GPU 运行时的创建、模型加载
- 模型推理任务仍主要通过预留保持活跃的预加载函数对用户请求进行响应，造成了大量计算资源的浪费。
  - 仍然需要为这些预先分配的 GPU 资源进行付费
  - （幂律定理）大多数的函数展现出较低的请求速率，约 85% 的函数请求速率为 1 req/min
- 采用常见的 GPU 共享方式：请求级别的 GPU 共享
  - 将多个模型放置在一张 GPU 卡中，但空闲的模型仍然会导致 GPU 显存饱和，无法充分利用 GPU 的计算资源
  - 单卡服务多个模型可能导致短时间内的服务过载，造成 GPU 热点和负载不均衡（可以采用模型并行的方式进行弥补？）

机器学习推理任务的主要需求：

- Compliance to latency SLOs：企业用户通常有严格的在线推理时延要求，如 99%的请求需要在 200ms 内完成
- Pay-per-GPU-use：当函数被调用并且在 GPU 上运行才进行收费，从而实现用户成本的节省
- GPU-efficient inference：相比于其他资源，GPU 计算需要更大的成本，提高 GPU 的利用率可以基于少量的 GPU 数量完成更多的函数推理请求
  - 这需要细粒度并且高效的 GPU 共享机制

本次工作的主要 idea：基于模型交换（model swapping）加速模型推理的吞吐量

- 在 host memory 中保证函数的活性，并且可以增多服务的模型数量
- host memory 拥有更大的空间并且相比 GPU memory 更加廉价

FaaSwap 的实现：

- 利用异步的 API 重定向，减少函数和 GPU 之间的同步开销
- 通过流水线执行模型交换和模型推理，隐藏模型交换的开销
- GPU 内存的管理机制
  - 跟踪模型执行过程中的地址，交换后直接调整 CUDA API 的执行地址
  - 仿照机器学习框架的内存布局，减少内存社情开销

FaaSwap 的调度机制：

- 交换调度算法：将模型分为重模型和轻模型两类，重模型主要在 GPU 之间进行交换，轻模型在 host 和 GPU 之间进行交换
  - GPU 和 host memory 通过 PCIe 进行数据交换，GPU 之间通过 NVLink 进行交换
- 模型换出算法：尽可能将唯一的重模型保留在 cache 中，将轻模型或者在 GPU 中有多个副本的重模型进行换出
- 请求调度算法：将更容易完成 SLO 的作业优先调度，以尽可能提高符合 SLO 的函数数量

实现 model swapping 的挑战：

- 透明性和隔离性
- 模型在主机和 GPU 之间的 swapping 导致 bandwidth contention，需要具体设计 GPU 的调度和内存管理策略

FaaSwap 的系统架构：

- cluster manager：负责集群级别的任务，包括请求路由、节点申请、资源扩展
- worker node：functions、GPU server、intra-node router
  - GPU server：在每个 worker node 上运行，管理本地的 GPU 资源
  - model repo、executor

FaaSwap 面向下述的几个方面进行了相关的工作：

- 如何使能高效的远程 GPU 调用
- 如何获取模型的信息并实现低延迟的 model swapping
- 如何管理 GPU 内存
- 如何保证隔离性和容错机制

关键在于如何快速完成 model swapping：

- 在 host-to-GPU 的路径上，使用页锁定内存减少内存拷贝的开销，通过 PCIe 和 DMA 进行传输
- 在 GPU 之间使用 NVLink 进行数据传输，加快数据传输速率
- 为了保证函数访问的透明性，需要管理和翻译 GPU 的内存地址
- 由于模型推理大多通过各层的前向传播完成，因此可以在上一层的计算和下一层的传输通过流水线执行
