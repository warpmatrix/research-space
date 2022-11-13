# 龙蜥峰会eBPF技术

## SysAK 系统运维工具集 与 系统监控场景应用

SysAK (System Analyse Kit)：系统监控、系统诊断、系统介入

系统运行模式：监控模式、诊断模式

- 监控模式：系统资源指标、系统瓶颈指标、系统干扰指标、容器资源指标
- 诊断模式：用户场景、系统专项诊断
  - 用户场景：系统负载分析、IO 问题
  - 系统专项诊断：系统通用、调度/内存/网络/IO 子系统

开发框架：通用库 -> 工具编译框架

### 监控场景运用

监控服务 mservice：资源监控、异常告警、根因分析

增强监控指标：

- 计算资源：负载、容器运行任务数、容器阻塞任务数
- 内存资源：（全局/容器内）存资源回收延时分布、内存规整延时分布
- IO 资源：容器读写等待时间/排队个数/读写字节
- 应用运行抖动：
  - 抖动问题原因：进程、线程调度延迟、（软）中断响应不及时、内核态执行过长
- 系统健康宕机告警：调度延迟、内核态锁竞争、syscall 耗时、内存回收延时

[gitee 源码](www.gitee.com/anolis/sysak)

平台插件化：SysOM组件、云监控组件、Prometheus 组件

## eBPF技术的发展与挑战

一些介绍 eBPF 的 links：

- https://cloud.tencent.com/developer/article/1749470
- https://ebpf.io/zh-cn/
- https://zhuanlan.zhihu.com/p/378766217

eBPF 抓取在内核中抓取数据包：

- 存储造型、状态信息、对事件的处理函数

eBPF 对 linux 内核的影响：

- user-mode applications -> kernel-mode application
  - 经过验证注入内核执行
- system calls -> BPF Helper Calls

Modern Linux: Event-based Applications

- mircrokernel-ish: 以 BPF 的形式放置在核外

eBPF 的项目：系统安全、可观测、网络、BPF 的可使用性

eBPF 的机遇与挑战：

- 内核可编程：BPF 与 CPU 调度器结合，使得用户可以接入 CPU 的进程调度
- eBPF 链接器、eBPF 与 wasm 等其他项目的 glue

eBPF 的挑战：offensive eBPF、kernel function as a service

## eBPF 安全特性解析

monitoring $\neq$ observerability：

- 通过 eBPF 获取内核信息，进行 pattern 识别实现安全功能

eBPF 的安全特性：程序沙箱化、侵入性低、透明化、可配置

- 内核源代码收到保护不被修改
- eBPF 程序的验证步骤保证资源不会程序阻塞
- 数据透明收集、自动配置收集策略

eBPF 的 Verifier：内存验证

云原生场景下的 eBPF：

- 运行时的监控
  - 基于 eBPF 挂载内核函数编写过滤策略
- 在内核层对异常攻击直接进行响应

## 基于 eBPF 的程序摄像头的设想

云原生环境下的可观测性：troble shooting

- 程序执行留痕：提供可观测性
  - 经典可观测的方式：log、trace、metric
  - 需要根据人为想象、经验分析可能的痕迹

trace profiling：通过程序摄像头对线程级别得到执行的相关信息

- OnCPU、OffCPU 与 trace 关联形成的程序摄像头
- 线程 tracing 与资源消耗匹配形成对应 metric

可观测性领域的使用场景：锁占用占比事件、线程池、java GC、网络问题、文件读写

## CoolBPF 的应用实践

1. 远程编译
2. 本地编译核基础库封装
3. 低版本内核支持
4. BPF 自动生成核发布
5. eBPF 自动生成核发布

## Eunomia-bpf: eBPF 轻量级开发框架

[github](https://github.com/eunomia-bpf/eunomia-bpf)、[zhihu doc](https://zhuanlan.zhihu.com/p/558733871)

- 编写内核态即可运行
- 部署时不需要重新编译，编译与运行完全分离
- 标准化的分发方式：json/WASM 的分发
