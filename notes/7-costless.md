# Costless: Optimizing Cost of Serverless Computing through Function Fusion and Placement

[IEEE/ACM Symposium on Edge Computing SEC (2018)](https://www.computer.org/csdl/proceedings-article/sec/2018/944500a300/17D45Xh13uq)

论文的主要思路：考虑 serverless function 组成 workflow (pipeline) 的情况下进行  fuse multiple functions，避免在函数之间转移导致多次调用的开销

论文处理的主要问题：

- Function Fusion Problem：不同的函数可能需要不同的内存大小，需要在不同函数之间的内存大小以及执行时间进行 trade off
  - 如果任意融合函数导致总函数需要的内存大小取决于所需内存最大的函数，造成执行成本的增加
- Function Placement Problem：利用 Greengrass 服务，将算力要求小的问题放置在边缘设备上进行处理

问题建模方式：将 function fusion 和 function placement 转换为 cost graph 进行表示

- 主要处理对象
  - 并行 function 融合：时间取最大值、内存大小取和
  - 串行 function 融合：时间取和、内存大小取最大值
- 主要困难：需要处理两种 cost 的图最短路问题，无法直接使用经典的图最短路算法
- 问题的处理方式：建模为受约束的最短路问题 (Constrainted Shortest Path, CSP)，使用 LARAC 算法进行近似求解
  - 时间复杂度：$O(|E|^2\log^2(|E|))$

实验效果：

- 不同的 fusion 和 placement 的组合价格预估基本准确
- 不同的 fusion 和 placement 的组合执行时间预估略有差异
- 求解 CSP 问题在 execution time threshold 有一定宽限的情况下能够达到暴力搜索所有 fusion 和 placement 情况的最优解

论文的主要特点：考虑了不同函数之间的 fusion 和 placement，使得 serverless 执行有了更大的探索空间，因此可以获得更好的 delay 和 cost 的表现

<!-- TODO: learn about LARAC -->

Q&A：当涉及的函数较多时，可能导致 cost graph 规模较大
