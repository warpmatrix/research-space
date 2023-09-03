# FAASM: Lightweight Isolation for Efficient Stateful Serverless Computing

现有的云平台大多使用临时 (ephemeral)、无状态(stateless) 的容器作为隔离机制

现阶段仍然面临两大挑战：

- data access overheads：无状态的容器意味着运行应用访问数据必须付出额外的开销
  - 要么需要外部维护程序所需的状态，要么在函数之间进行状态的传递
  - 本质上是将数据移动到计算的模式，与大数据处理的模式相违背
- container resource footprint：使用传统的容器技术仍然与 faas 的大量短瞬的 workload 不匹配
  - 冷启动依然需要数百毫秒的启动时延
  - 容器的高内存占用是导致容器无法进一步扩展的主要原因，一台 16GB 内存的机器仅支持大约上千个容器的运行

针对上述的挑战，目前的解决方案有：

- 在长期运行的 VM 中维护 faas 所需的状态避免数据的移动，这样引入了非 serverless 的组件
- 重用容器，将多个函数运行在同一个容器中，这种做法实际上削弱了函数的隔离保障
- 引入静态虚拟机同样打破了 serverless 的范式，并且静态虚机需要随着函数数量进行扩展
- 即使使用 local storage 减少数据访问的开销，但均没有实现共享内存
  - 仍然需要在不同的函数进程的内存空间中复制数据，没有解决需要进行数据传输的问题，带来了网络和内存的开销

本文提出的 Faasm 具有以下特性：

- lightweight isolation 通过 cgroups, CFS, network namespaces 和 traffic shaping 实现 software fault isolation
- local/global state access: 面向 in-memory 和 distributed 两种情况分别设计 local/global tier 访问数据
- fast initialisation: 设计 Proto-Faaslet 进行预实例化，实现跨主机恢复的快速启动
- flexible host interface

## 主要工作

提出 faaslet 作为隔离机制：

- 通过 webassembly 实现了 private memory 和 shared memory，并且利用 webassembly 实现内存的强隔离
- 通过 linux cgroups 和 linux CFS 实现 CPU 资源的隔离和调度
- 通过 network namespaces, traffic shaping 实现网络资源的共享和隔离

基于 webassembly 实现一系列虚拟化接口：

- 这些接口实现了函数与主机之间内存安全的交互，能够动态链接函数运行代码、完成主机实现接口 (trunk) 的注入
- 实现的接口包含：函数调用、状态访问、动态链接、内存管理、网络、文件IO

在 process memory 中实现 shared regions 以完成函数间的数据共享：

- faaslet 使用 `mmap` 和 `mremap` 进行内存扩展，通过 flag 标识是 new private memory 还是 shared memory
- 提出了 distributed data objects 统一 local tier 和 global tier 的共享数据访问接口
- 在 global tier 中使用 key-value store 来实现数据的索引
- 通过 local read/write lock 和 global read/write lock 保证 local tier 和 global tier 的数据一致性

faasm runtime：基于 faaslet 实现分布式调度

- 尽可能将应用请求调度到 warm faaslet 中（如果本机可以热启动或存在其他热启动主机）
- 在 global tier 中保存 warm hosts set
- 通过 Proto-Faaslet 进行沙箱的重置，减轻冷启动带来的时延、并且实现多租户的隔离
  - 将每次调用都执行的代码称为 initialisation code，用于生成每个函数特定的 proto-faaslet
  - proto-faaslet 快照包含的组件：stack, heap, function table, stack pointer, data
  - 通过 copy-on-write 的方式进行快照的复用
- 每个 fssm 实例拥有的组件如下：local scheduler, pool of Faaslets, collection of state stored in memory, sharing queue

faaslet 生成可执行文件的过程：

1. compilation (untrusted): source -> webassembly
2. code generation: webassembly -> `.obj` file
3. linking: `.obj` file + host interface (inject) -> executable
