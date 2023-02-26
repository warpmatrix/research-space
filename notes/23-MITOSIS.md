# No Provisioned Concurrency: Fast RDMA-codesigned Remote Fork for Serverless Computing

OSDI'23

将多个机器使用容器等方式，抽象成单个机器向用户提供服务，无需指定到底需要运行多少台机器

- start、run、kill，scale on demand
- cold start：docker pull、init container、init runtime
- 冷启动的时间占据大部份的容器运行开销，67%@Edge 少于 20ms 的运行时间，冷启动秒级时延

remote fork 的优点：

- efficient in both performance and resource usage
- provide transparent intermediate state sharing between remote functions

业界 warmstart 的解决方案：

- cache ：find a cached container (pause)、unpause container
  - 需要预置 $O(n)$ 的热容器数量，云供应商无法很好地预估请求到达数量，需要用户自己预估配置热容器的数量
  - 消耗大量内存资源降低利用率，在启动速度和内存利用率中进行 trade-off
- fork：每台机器保留 $O(1)$ 的容器数量作为模板，供每一台机器 fork 出新的容器
  - 使用多个机器进行 fork 操作完成容器扩容，总共只需 $O(m)$ 的资源数量
- checkpoint：利用文件保存容器 checkpoint 数据，并且利用分布式文件系统进行分发和 restore
  - 总共只需保留 $O(1)$ 的资源，但 restore 的速度远远慢于 cache 和 fork
  - 可以使用 RPC 避免 checkpoint 操作 -> RDMA
  - fork 操作涉及用户数据，只能在同一个应用中进行或者使用操作系统接口进行限制

状态传输开销：

- copy data via message passing
- 云上存储服务进行数据交换

MITOSIS：利用 RDMA-caoable NICs (RNICs)，模仿本地的 local fork 行为

实现与主要工作：

- 原语：fork prepare、fork resume
- 网络连接：DCQP、DCT，减少网络连接的开销适应 ms 级别的冷启动时延读写
- 内存管理：缺页处理、最差退化为 RPC、基于连接的内存访问控制、预取与缓存内存
- 采取 copy-on-write 的方式直接读取 parent memory

fork for serialization-free state transfer:

- 数据传输方式：message passing or cloud storage
- 使用 serialization-free 的方式减少开销
- pre-fetch 和操作系统抽象在 RDMA 中的实现

limitation：

- 需要利用现有的技术启动第一个容器，才能进行后续的 fork 操作
- fork 只允许只读的状态传输，能 cover 住 serverless 大部分需求
- 当存在多个上游函数时，需要特殊处理融合为一个函数或者使用消息传递的方式处理
