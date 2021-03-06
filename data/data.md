# 扫地机物体识别项目总结

## 物体检测

### 图像采集
* 原则上最好用高清彩图,可以减少光照等影响.
* 采集用的相机型号最好和测试场景使用相机型号一致,保证视角类似,相机高度类似.
    相机过低对小物体影响挺大，侧面看分不清类别.
* 最好在扫地机跑起来的时候采集图像，提前摆好物体即可.
    这样既方便采集又比较贴合测试时情况
* 类别选择, 对于人这类无需过多标记，开源数据集里已有很多，也可以识别的很好.
* 可以提前做好图像预处理，比如直方图均衡，方便标记，当然测试时也最好做同样的操作



### 抑制背景(误检)
* 实际测试中少不了背景框的产生，特别是对于没见过的场景,故可以按照
以下步骤来处理。

1. 收集产生背景框的图片然后标注背景框（框的位置可以随便标记，类别必须标为bg）
2. 训练代码中每个batch输入中必须保证背景样本的数量。
    ```python
    neg_samples = min(min_neg_samples, neg_samples)
    ```
不断迭代直至背景完全消失。

注意对于漏检也可采取同样的做法，对识别错误的样本进行标注加入训练集即可


### 标注
* 目前统一采用labelme 开源工具标注，使用内部开发的工具转化为统一的hdf5数据格式
标注前确定一份classes.txt，指定所需标注的物体名称即可,无需按照顺序（类似于bg, bed）

* 标注所有看的清的物体(足够大,size_ratio>8/300)，部分露出来也需要标注,但露出部分过少则不标记
训练的时候可以自行忽略过小的样本


### 深度学习框架
* 目前用开源实现的SSD即可,得首先测试能否转到RK3399上GPU运行，如果模型过复杂可能需要
自己实现opencl算子，故有两种方案
1. 详细了解MNN框架，为其实现算子,需要理解MNN的设计
2. 自研inference框架，只实现公司所需算子,目前正在开发中


### 持续集成(CI)
* 模型免不了reversion，到底得重新训练整个数据集还是增量训练这也是个问题
1. 目前仍采取的是重新训练整个数据集的方法


## 物体定位
* 根据识别2d框估计物体的位置，达到避的目的

### 深度估计
* 一些方法和其注意事项
1. 相机俯仰角会对一切依赖地平面约束的方法产生影响, 要么保证水平要么测量俯仰角进行计算
2. 要么利用物体实际大小进行测量,需要大概已知物体实际大小
3. 深度学习估计深度,需要建立深度数据集，需要深度标注

### 稳定性
* 不同位置估计物体会产生不一样的结果造成位置跳变，目前采取的方法是取物体最近距离的估计结果为准
同时采用多帧结果融合为准,多帧检测出来才算正样本.

### 边界处理
* 图像边界上由于物体部分可见，可能没检测出来, 故对处于边界物体不进行消除逻辑处理,
对于部分可见造成的位置不准导致可能匹配不上的情况,可以适当进行修正，并在完全可见时进行更新
