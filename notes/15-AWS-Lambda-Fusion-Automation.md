# Mitigating Cold Start Problem in Serverless Computing with Function Fusion

[Sensors'21](https://www.mdpi.com/1424-8220/21/24/8416)、[src](https://github.com/henry174Ajou/AWS-Lambda-Fusion-Automation)

冷启动在 serverless workflow 中的表现（之前的文章主要通过一个函数作为角度进行论述）：

- 冷启动随着函数链的增长不断累积
- 冷启动的时延可以占据 workflow 响应时间的一半左右

多个函数协作的方式：反射调用、事件驱动、函数融合

函数融合的优势：

- 融合后的函数和预加载的效果一致，可以减少冷启动
- 无需引入 reflective invocation 中使用的 scheduler function
  - scheduler function 和工作流中的函数并行执行，存在重复收费的问题
- 无需像 trigger method 中一样需要修改原函数，添加发射事件的代码

函数融合中存在的一些问题：

- 融合后函数的执行时间可能超过 serverless 平台的最大执行时间
- 融合后的函数内存利用率下降，增大函数执行的成本

一些很奇怪的观察：

- serverless 申请更大的内存，导致冷启动的时间减少
- 更大的代码导致冷启动所需的时间减少

产生冷启动的两种情况：函数资源被回收、函数的自动扩展

workflow 不同的组建条目：

- function：workflow 具体工作内容的逻辑实现
- fan-out：map，数据并行执行同一个 map 程序
  - 无服务器平台可以自动扩展完成并行计算
  - 融合后可能牺牲并行性，通过预测融合前后函数执行时间作为是否融合的判断依据
- condiction branch
  - recursive function fusion 的体现方式之一
  - 存在分支的 workflow 完成分支内的 function fusion 才进行上层 workflow 的 function fusion

实现的思路：使用 mediator function 作为函数之间的协同和数据传输

- mediator function 内直接调用函数模块，没有创建新的 lambda function
- 不会导致 double billing 的问题
- 每个 workflow 条目都有自己是否融合的对应标准，第 4 节的内容

system 的具体实现：可以进一步考虑 data locality 和 provision 提高 fusion decision 的智能化
