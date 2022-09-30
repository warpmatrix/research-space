# Spatula: Efficient cross-camera video analytics on large camera networks

[ACM/IEEE Symposium on Edge Computing (SEC 2020) - microsoft research](https://www.microsoft.com/en-us/research/publication/spatula-efficient-cross-camera-video-analytics-on-large-camera-networks/)

论文主要思路：在大规模的摄像头网络中，利用摄像头之间的时空关系进行跨摄像头的视频分析

- 时间关系：跨帧分析；空间关系：跨摄像头分析
- 研究方向：利用时空关系进行剪枝，减少查询物体的搜索空间，达到效果计算开销和出现物品的摄像头数量成正比
- 关键性质：目标检测物在某一时刻只出现在部分摄像头中（视频分析中存在大量低价值的冗余数据）
- 此前的研究：针对单一视频流进行优化（降采样、使用滤波器丢弃无用帧），优化方法没有结合其它视频流进行分析

主要挑战：

- 难以获得摄像头之间的时空关联开销过大
- 需要系统支持对视频流进行抽象，并且抽象简洁有效
- 若目标丢失需要系统快速进行修正

系统运行的步骤：

0. offline profiling phase：离线构建高开销的时空关系模型，提前检测跟踪实体将时空信息聚合到模型中
1. inference time：
