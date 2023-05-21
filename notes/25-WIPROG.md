# WIPROG: A WebAssembly-based Approach to Integrated IoT Programming

[infocom'20](https://ieeexplore.ieee.org/document/9488424), [github](https://github.com/liborui/WiProg)

相关背景：

- 云边端协同的场景下，可以使用同一种连贯的语言进行编程，但依然会存在一些问题：
  - 由于不同环境运行时的差别 (linux on cloud and edge, arduino on device)，无法无缝进行代码的迁移
  - 同时也限制了代码迁移带来的性能提升效果
- 如果采用传统的方式实现代码卸载，需要手工实现应用程序的逻辑分区和设备之间的数据流
- WebAssembly 的三种机制：linear memory model、table object、import mechanism

主要工作：

- 通过外设访问接口和基于属性的注释进一步编译为 webassembly 程序
  - 通过预定义的约束检查函数注释的合法性，并且进一步分配没有进行注释的函数
- 程序卸载步骤：通过卸载策略和性能侧写完成是否卸载的决策、通过中断打断正常的执行流记录现场、使用 RPC 调用远程函数

主要解决的问题：

- 拦截 WebAssembly 模块的执行流进行 RPC
  - 构造辅助函数实现函数上下文的捕获和恢复
  - 执行主机中导入 function table
  - 完成 `call` 到 `call_indirect` 的替换，在运行时将运行函数分发给 function_helper
- 远程调用函数中状态和参数的打包和拆包
  - 基于紧凑内存快照机制，减少网络传输开销，只在卸载的时候捕获必要的数据

## reference

- the performance degradation of the JavaScript engine compared with the native approach on IoT devices is over 100× [6].
- we conducted preliminary experiments comparing WebAssembly with the native and the existing cross-platform languages (C#, Java) used by state-of-the-art computation offloading approaches [3]
- We use five benchmarks from the Computer Language Benchmarks Game (CLBG) [10]
- capture the environmental information (i.e., bandwidth, round-trip-time) to facilitate the intelligent decision of offloading policies, which is similar to [13]
- digital I/O, analog I/O, and communicationrelated ones (e.g., UART) are commonly used and cover 89.78% of the total APIs we have investigated [19]
- To achieve remote execution, the most recent work on WebAssembly offloading, MWW [4]
- existing partial runtime snapshotting approaches [2], [20] which directly packs the function stack for offloading
