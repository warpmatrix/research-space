<!-- omit in toc -->
# Performance Optimization for Edge-Cloud Serverless Platforms via Dynamic Task Placement

背景：

- for smart edge devices, for example, smart cameras or speakers
- need to perform processing tasks on input data in **real to near-real time**

数据来源：AWS Lambda and AWS Greengrass

效果：以较低的预测误差，降低 3 个数量级的延迟（与仅使用边缘执行相比）

<!-- omit in toc -->
## Table of Contents

- [1. 概述](#1-概述)
- [2. 流水线](#2-流水线)
  - [2.1. 云端流水线](#21-云端流水线)
  - [2.2. 边缘流水线](#22-边缘流水线)
  - [2.3. 应用](#23-应用)
- [3. 模型框架](#3-模型框架)
- [4. 性能评价模型](#4-性能评价模型)
  - [4.1. 云端性能模型](#41-云端性能模型)
  - [4.2. 边缘性能模型](#42-边缘性能模型)
  - [4.3. 模型训练](#43-模型训练)
- [5. 框架实现](#5-框架实现)
  - [5.1. 预测器实现](#51-预测器实现)
  - [5.2. 决策引擎](#52-决策引擎)
- [6. 实验结果](#6-实验结果)
  - [6.1. 最小成本问题](#61-最小成本问题)
  - [6.2. 最小延迟问题](#62-最小延迟问题)

## 1. 概述

提出原因：

- 数据量的激增对于网络资源提出较高的要求
- 许多应用对延迟敏感，需要接近实时地访问数据资源

边缘优势：利用边缘的计算资源，对数据进行初步处理，得到较小大小的结果，再上传到网路

根据不同计算任务的种类，可以在边缘端进行计算，也可以将计算任务分配 (offload) 给服务器

两个对称的问题：

1. 满足开销约束的条件下，最小化延迟
2. 满足延迟约束的条件下，最小化开销

延迟的定义：输入的摄入到在云上存储结果

## 2. 流水线

### 2.1. 云端流水线

冷启动与热启动的概念：

- cold start: If a new container needs to be created before executing a function, this
- warm start: If a function is assigned to an existing container

端到端延迟的计算 (performance)：$T_c(k) = upld(k) + start(m) + comp(k, m) + store(k)$

- 数据上传时间
- 容器启动时间
- 计算时间
- 存储数据时间

价格开销 (cost)：与容器内存大小和执行时间成正比，并且每次查询收取固定费用

### 2.2. 边缘流水线

边缘使用 a long-lived function 的原因：

- 没有足够的资源、能力并行运行
- 多函数竞争资源产生难以预测的结果

端到端延迟 (performance)：$T_e(k) = comp(k) + iotup(k) + store(k)$

- 计算时间
- 数据上传时间
- 云端存储数据时间

流水线价格开销 (cost)：按年计费，分摊到每一次的查询费用为零。

### 2.3. 应用

针对不同的应用场景，设置不同的输入速度。

- Image Resizing
- Face Detection
- Speech-To-Text

## 3. 模型框架

- Decision Engine
- Executor
- Uploader

需要解决的两类问题：

- 满足延时的需求，价格最小
- 满足价格的需求，延时最小

    增加的考虑因素：surplus 成本（unused budget），$\text{subject to. } C(k) \leq Cmax + \alpha \times surplus(k)$

## 4. 性能评价模型

### 4.1. 云端性能模型

- $upld(k) = θ_1 + θ_2 × size(k)$
- $start(m)$ 遵循正态分布
- compute time：gradient boosted regression
- store time：输出通常较小，且比较相似。使用训练数据的期望从 normal random variable 中预估
- Container idle time：与输入和应用特征无关。用一个值进行模拟，使用二叉树进行搜索（得到 27 min）。

### 4.2. 边缘性能模型

- $comp(k) = \Phi_0 + \Phi_1 × size(k)$
- $iotup(k)$ 得到的结果较小，使用训练数据的均值对应的正态分布进行预估
- $store(k)$ 使用训练数据的均值进行估计

### 4.3. 模型训练

云端模型需要各自收集冷启动和热启动的数据。热启动需要使用无用输入运行函数。冷启动时间与容器的内存大小无关。

由于可能的网络波动，预测云端模型的延迟可能不准确，云端流水线的延迟本身具有较大的波动。

## 5. 框架实现

### 5.1. 预测器实现

已有空闲的容器用于收集热启动的时间；若没有容器或所有的容器都忙用于收集冷启动时间。

每次预测完成后，都调用 `updateCIL` 方法更新数据。

### 5.2. 决策引擎

根据具体的问题选择最合适的云端配置或边缘执行

注意到如果在最小价格问题中，无法找到合适的配置，在边缘执行的价格为 0。

## 6. 实验结果

### 6.1. 最小成本问题

1. 成本预测误差小，能够让框架实现更小的总成本开销
2. 当任务的执行时间足够小时，一个小的执行时间误差可能导致较大的成本误差
3. 更少的时间违反与总成本开销具有良好的性能相关

关于 Face Detection 的问题：as the deadline increases, the number of edge executions decreases, and accordingly, the cost increases. This is because, with a larger deadline, tasks are assigned to the edge, causing the edge to be busy, which in turn, leads to more tasks being assigned to the cloud

更小的输入速率，减少错误判断冷启动和热启动的概率

### 6.2. 最小延迟问题

$C_{max}$ 和 $\alpha$ 足够小，因此有必要使用边缘端进行执行 $λ_{edge}$

1. $\alpha$ 的增加可以减少端到端的延迟
2. 当 $\alpha = 0$ 时，端到端的延迟比较高
3. $C_{max}$ 较小 $\alpha$ 较大时，可能导致超出预算
