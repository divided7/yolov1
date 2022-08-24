# yolov1
楼下下棋大爷都能看得懂的yolov1！


#### 1、模型功能
![model](https://github.com/divided-by-7/yolov1/blob/main/image/yolov1.jpg)
一句话概括：任意形状的RGB 3通道图片[3,H,W]，经过yolov1后，得到一个[30,7,7]的输出。
那这个输出是如何变成图像上的一个个框呢？

yolov1的思想是，把原图划分成7x7的区域，在每个区域上都以区域的中心为anchor center，预测两个框，这两个框是可以很大，甚至超出本区域的，核心思想是本区块为中心的框。最终，一定能得到7x7x2个框。but：(1)实际上从没有见过这么多框？这是因为不仅仅预测了框的位置，还预测了框里的内容是物体（object）还是背景。如果是背景，那么这样的框就不会绘制。(2)如果相邻的两个区块都很贴近物体，那预测的框不是会几乎重合吗？为了解决这个问题，在最后还有一个非极大抑制（NMS）的操作，对同一类别IOU较大的两个框做抑制，即删去置信度低的框。

输出特征图的第2、3通道的[7,7]代表原图[H,W]特征提取的特征图，可以理解为将输入图片均匀划分成7×7个区块，每个区块为[H/7,W/7]个像素。那么输入的每个区块都对应了一个30个数据构成的输出。
这30=20+4+4+1+1个数据有20个数据代表的是类别，8个数据代表两个框，分别是框的(x,y,w,h)，2个数据代表这两个框的置信度。

#### 2、模型的前向传播：
输入图片,shape=[3,H,W] -> reshape成[3,448,448] -> 使用GoogLeNet的前面几层，做32倍下采样，即高和宽缩小32倍，形状变成[1024,14,14] -> 经过一些卷积层激活层后，形状变成[1024,7,7] -> flatten成一维向量，形状为[50176] -> 使用两层线性层（也称全连接层），形状变成[1470] -> reshape成[30,7,7]

#### 3、模型的反向传播：
主要就是损失函数了。输出的30个通道，其包含：20个分类信息、（4个框的xywh位置信息）×2、（一个框的置信度）×2。对于分类信息的误差，选用的是MSE。为什么不用交叉熵呢？可能这就是时代的变迁吧；对于置信度的损失，就是一个数字的预测，看起来好像没啥好说，直接MSE？NoNoNo！首先先问问自己，预测出来的置信度好说，真实的置信度该是多少呢？这里的置信度值为预测的框和真实框的IOU值，且原文还将是否有物体的情况拆分开，在无物体的情况下MSE还加了一个超参数权重，用于修改样本比例。xy则是使用典型的sum[(x-hat{x},y-hat{y})^2]，wh和xy的公式差不多，只不过对w和h开了根号，目的是为了让比较大的框有较小的更新，比较小的框有较大的更新。
![bp](https://github.com/divided-by-7/yolov1/blob/main/image/3.jpg)

# 分割线

如果没有读博的话，可能有时间会闲着没事写点代码玩玩，不过v1相当于是上古时代的宝贝了，也就是自己玩玩，和yolo系思想学习，没有实际的使用价值。
