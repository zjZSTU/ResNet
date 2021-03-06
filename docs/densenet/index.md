
# DenseNet

参考：[学习 Densely Connected Convolutional Networks](https://blog.zhujian.life/posts/4129fe4c.html)

`DenseNet`通过密集连接设计，将每一层输出作为后续所有层的输入，通过加强特征传播，能够减轻梯度消失的问题；同时通过鼓励特征重用，大大减少了参数数量

![](./imgs/figure-2.png)

`DenseNet`主要由两个模块组成：

1. `Dense Block`（负责特征提取）
2. `Transition Layer`（负责特征图衰减）

其中`Dense Block`中使用$3\times 3$卷积进行特征提取，同时利用$1\times 1$卷积来控制输入特征图个数，所以组成一个`Dense Layer`模块（`Conv(1x1) + Conv(3x3)`）进行实现

## Dense Layer

* 实现文件：`py/lib/models/densenet/dense_layer.py`

其实现流程如下：

```
BN -> ReLU -> Conv(1x1) -> BN -> ReLU -> Conv(3x3)
```

##  Dense Block

![](./imgs/figure-1.png)

* 实现文件：`py/lib/models/densenet/dense_block.py`

在每个`Dense Block`中，由多个`Dense Layer`组成，同时其每个`Dense Layer`的输出都作为后续`Dense Layer`的输入。不同的输入数据通过连接（`concatenating`）方式合并

## Transition Layer

* 实现文件：`py/lib/models/densenet/transition.py`

通过`Trantition Layer`模块进行特征图减半操作，每个`Tranisition`由一个`Conv(1x1)`和一个$2\times 2$大小的平均池化组成，其实现流程如下：

```
BN -> ReLU -> Conv(1x1) -> AvgPool(2x2)
```

## DenseNet

参考：[pytorch densenet.py](https://github.com/pytorch/vision/blob/master/torchvision/models/densenet.py)

* 实现文件：`py/lib/models/densenet/denset_net.py`

![](./imgs/table-1.png)

## 训练

![](./imgs/figure-3.png)

比较`DenseNet-121`和`ResNet-34_v2`

* 数据集：`voc 07+12`
* 迭代次数：`100`
* 批量大小：
    * `48（train）`
    * `48（test）`
* 图像预处理：
    * 训练：缩放+随机裁剪+随机水平翻转+颜色抖动+随机擦除+数据标准化
    * 测试：缩放+`Ten Crop`+数据标准化 

* 损失函数：标签平滑正则化，平滑因子`0.1`
* 优化器：`Adam`，学习率`3e-4`，权重衰减`3e-5`
* 学习率策略：`warmup`（共`5`轮）+余弦退火（`95`轮）

## 训练结果

![](./imgs/loss.png)

![](./imgs/top-1-acc.png)

![](./imgs/top-5-acc.png)

完整训练日志参考[训练日志](./log-dense101-vs-resnet34_v2.md)

### 检测精度

* `Top-1 Accuracy`
    * `DenseNet-121:  89.86%`
    * `ResNet-34_v2: 90.50%`
* `Top-5 Accuracy`
    * `DenseNet-121: 99.20%`
    * `ResNet-34_v2: 99.29%`

### Flops和参数数目

```
densenet_121: 5.731 GFlops - 30.437 MB
resnet-34: 7.349 GFlops - 83.177 MB
```

## 小结

| CNN Architecture | Data Type (bit) | Model Size (MB) | GFlops （1080Ti） | Top-1 Acc(VOC 07+12) | Top-5 Acc(VOC 07+12) |
|:----------------:|:---------------:|:---------------:|:-----------------:|:--------------------:|:--------------------:|
|   ResNet-34_v2   |        32       |      83.177     |       7.349       |        90.50%        |        99.29%        |
|   DenseNet-121   |        32       |      30.437     |       5.731       |        89.86%        |        99.20%        |

从训练轨迹上看，`DenseNet-121`在前期能够得到更快的收敛速度，在后期两者逐渐趋同。`DenseNet-121`比`ResNet-34_v2`拥有更小的模型和`Flops`，两者也能够训练得到相近的准确度

进一步训练方向：

1. 更大批量训练
2. 更多数据集训练
3. 使用预训练模型