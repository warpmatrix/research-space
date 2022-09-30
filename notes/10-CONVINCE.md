# CONVINCE: Collaborative Cross-Camera Video Analytics at the Edge

[2020 IEEE International Conference on Pervasive Computing and Communications Workshops (PerCom Workshops)](https://ieeexplore.ieee.org/document/9156251)

CONVINCE: **CO**llaborative **N** Cross-Camera **VI**deo a**N**alyt**C**is at the **E**dge

论文的主要研究方向：针对有重叠区域（Field of Views, FoV）的摄像头，研究如何应用时空关系降低开销和提高性能

- 基本动机：一些摄像头可能由于位置、天气等因素，得到的图像分辨率较低，可以通过其它摄像头的信息作为补充提高性能
- 将边缘端的摄像头网络看作是实体集合，结合边缘服务器将计算任务下方到边缘侧
- 在本地进行处理消除冗余帧，减小网络开销并提高应用精度

系统的两种实现方式：

- 集中式实现：使用带有计算能力和存储能力的边缘服务器
  - 计算深度的视频处理任务，管理摄像头之间的时空关系、拓扑关系
  - 可以使用联邦学习的技术解决隐私问题
- 分布式实现：每一个摄像头都带有和其它摄像头的交流能力和拓扑信息
  - 可以使用不同的交流策略：peer-to-peer channel、all-to-all channel
  - peer-to-peer channel：和相关（含有相同重叠区域或追踪相似目标）的摄像头进行通信
  - 需要在信息传递和推理时延中进行 trade-off

评价指标：模型的准确度、摄像头之间传输数据的帧数

一些需要解决的挑战：

- 如何应对对抗节点（adversarial node），解决方法如引入信任评价机制对节点进行信任度打分
- 处理训练集数据规模小，解决方法如在不牺牲隐私的前提下使用其它摄像头共享的数据进行训练、使用少镜头学习的相关技术进行训练（数据增强、元学习）

一些可以继续研究的方向：

- 设计模型建模目标在摄像头之间移动的时空关联，如 spatula
- 如何自动识别和节点相关（可相互协同）的其它摄像头节点，在未来的发展中很可能出现可移动的摄像头。可能的解决方法如对摄像头进行时空关联的聚类操作
- 摄像头之间进行协作的隐私保护问题，可能关联的领域如视觉物联网（Internet of visual thing，IoVT）、人脸识别、表情识别、场景分析
