## 编程实现过程

1.  分析神经网络应该采用什么结构。主要涉及以下方面：

> *   问题域的内容如何处理。每个神经元处理的数据是什么。
> *   神经网络中的层之间的权重weight如何定义。
> *   神经网络层之间的阈值如何定义。

1.  初始化神经网络

### 经验

1.  从huggingface.co下载模型资源时失败
    错误信息：Error retrieving file list: (MaxRetryError("HTTPSConnectionPool(host='huggingface.co', port=443):
    解决方式1： 取消SSL。测试有效
```
    # 在python文件的开头添加这个
    import ssl
    ssl._create_default_https_context = ssl._create_unverified_context
```
解决方式2：取消SSL。未测试过

    '''This following code will set the CURL_CA_BUNDLE environment variable to an empty string in the Python os module'''

    import os
    os.environ['CURL_CA_BUNDLE'] = ''

## 资源

*   fastai -- AI库，推荐作为学习使用。包含很多的内容。
*   ImageNet -- 图片数据库，用于训练模型
*   模型resnet18 -- 图像识别，用ImageNet进行训练

### huggingface接入
添加国内镜像站，具体可访问 https://hf-mirror.com/
配置HF_HOME、HF_ENDPOINT
```
export HF_HOME=/mnt/ext_file/huggingface_home
export HF_ENDPOINT=https://hf-mirror.com      
```
安装huggingface_hub进行模型和数据的下载缓存
```
pip install -U huggingface_hub
# --local-dir会指定将模型下载到特定的路径下
huggingface-cli download facebook/opt-125m --local-dir facebook-opt-125m
# 下载数据集
huggingface-cli download --repo-type dataset --resume-download wikitext --local-dir wikitext
```

### modelscope下载模型
配置MODELSCOPE_CACHE设置下载模型的缓存路径，比如
```
export MODELSCOPE_CACHE=/media/zhuqingquan/project/code/models-cache/modelscope
```
安装modelscope sdk
```
pip install modelscope
```

## 能力点

| 方向      | 细分                                                 | Col3 |
| ------- | -------------------------------------------------- | ---- |
| 训练一个模型  | 1. 数据收集、过滤 <br> 2. 将数据拆分为训练数据与验证数据集 <br> 3. 数据如何标记 |      |
| 加载模型并推理 |                                                    |      |

## 数据

The intuitive approach to doing data cleaning is to do it *before* you train a model. But as you've seen in this case, a model can actually help you find data issues more quickly and easily. So, we normally prefer to train a quick and simple model first, and then use it to help us with data cleaning.

### 训练数据集 dataset

[The Oxford-IIIT Pet Dataset-宠物数据集](https://www.robots.ox.ac.uk/\~vgg/data/pets/)
[MNIST--手写数字]()

## 模型

#### ResNet

ResNet-34 from `Deep Residual Learning for Image Recognition` <https://arxiv.org/abs/1512.03385>
[pythorch中关于这个模型的训练和使用的一些代码](https://github.com/pytorch/vision/blob/main/torchvision/models/resnet.py)
类 ResNet是各种类型的ResNet模型的抽象基类。

#### ollama部署大模型

[github ollama](https://github.com/ollama/ollama)
```shell
    # 安装ollama
    curl -fsSL https://ollama.com/install.sh | sh
    # 启动ollama服务
    ollama serve
    # 安装并启动模型
    ollama run llama3.1
    # 查看正在运行的模型
    ollama ps
    # 停止模型
    ollama stop <模型名称>
    # 删除模型
    ollama rm llama3.1
    # 查看所有安装的模型
    ollama list
```

## 模型量化

### 创建量化网络有两种工作流程

训练后量化(PTQ: Post-training quantization) 在网络经过训练后得出比例因子。 TensorRT 为 PTQ 提供了一个工作流程，称为校准(calibration)，当网络在代表性输入数据上执行时，它测量每个激活张量内的激活分布，然后使用该分布来估计张量的尺度值。

量化感知训练(QAT: Quantization-aware training) 在训练期间计算比例因子。这允许训练过程补偿量化和去量化操作的影响。

### 量化网络可以用两种方式表示

在隐式量化网络中，每个量化张量都有一个相关的尺度（Scaler）。在读写张量时，尺度用于隐式量化和反量化值。
与隐式量化相比，显式形式在模型网络中显式插入Q、DQ层进行量化和反量化，显式形式精确地指定了与INT8之间的转换在何处执行，并且优化器将仅执行由模型语义规定的精度转换

|                             | Implicit Quantization                                                                                                             | Explicit Quantization                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| User control over precision | Little control: INT8 is used in all kernels for which it accelerates performance.                                                 | Full control over quantization/dequantization boundaries.                      |
| Optimization criterion      | Optimize for performance.                                                                                                         | Optimize for performance while maintaining arithmetic precision (accuracy).    |
| API                         | Model + Scales (dynamic range API)Model + Calibration data                                                                        | Model with Q/DQ layers.                                                        |
| Quantization scales         | Weights\:Set by TensorRT (internal)Range \[-127, 127]  Activations\:Set by calibration or specified by the userRange \[-128, 127] | Weights and activations\:Specified using Q/DQ ONNX operatorsRange \[-128, 127] |

TensorRT中PTQ功能可生成隐式量化网络。TensorRT可能在图优化期间将该层与另一个层融合，并丢失它必须在 INT8 中执行的信息。
TensorRT中隐式量化的2中方式：

1.  [Setting Dynamic Range](https://github.com/NVIDIA/TensorRT/tree/main/samples/sampleINT8API)
2.  Post-Training Quantization using Calibration

[tensorrt-int8量化的介绍](https://developer.nvidia.com/zh-cn/blog/tensorrt-int8-cn/)
[大模型性能优化（一）：量化从半精度开始讲，弄懂fp32、fp16、bf16](https://zhuanlan.zhihu.com/p/667163603)
[CUDA使用FP16进行半精度运算](https://v2as.com/article/2d08da78-0d41-40ed-a9c5-7886c31bd8ef)
[混合精度原理与计算过程（AMP）](https://www.hiascend.com/doc_center/source/zh/Pytorch/60RC1/ptmoddevg/trainingmigrguide/PT_LMTMOG_0083.html)
