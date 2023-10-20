## Windows安装
+ [安装NVIDIA Nsight Integration for vs stdio](https://developer.nvidia.com/nvidia-nsight-integration-install-tips)
+ [安装NVIDIA Nsight Graphics](https://developer.nvidia.com/nsight-graphics)
    NVIDIA Nsight Graphics用于调试GPU问题，也可以调试OpenGL的问题。
+ [NVIDIA Nsight Integration](https://developer.nvidia.com/nsight-tools-visual-studio-integration)
+ 安装后确认是否成功以及查看版本号
```
nvcc --version
```

## 安装cudnn（Windows）
[下载地址](https://developer.nvidia.com/rdp/cudnn-download)    ==注意需要下载与cuda对应版本的cudnn。==
[官方安装指引](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)
+ 注意：安装官方指引是将cudnn的include、lib、bin单独放在一个文件夹里面。但是其实将他们放在cuda的目录下对应的include、lib、bin下面会更方便一点。
    + include拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\include
    + lib拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\lib\x64
    + bin拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin

## 安装tensorrt(Windows)
+ GA与EA的区别
EA 版本代表抢先体验（在正式发布之前）。
GA 代表通用性。 表示稳定版，经过全面测试。
[下载页面](https://developer.nvidia.com/nvidia-tensorrt-download)
[下载页面-8x](https://developer.nvidia.com/nvidia-tensorrt-8x-download)==注意下载与cuda对应版本的tensorrt==
+ 直接解压拷贝include、lib、bin到cuda安装目录下
    + include拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\include
    + lib拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\lib\x64
    + bin拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin
+ [可以参考](https://zhuanlan.zhihu.com/p/339753895)
+ 以上安装的只是c/c++的tensorrt库。
+ python库的安装
    + 下载的tensorrt zip包中也包含python的whl安装文件。可以直接还用pip install安装
    ```
    cd TensorRT-8.6.1.6.Windows10.x86_64.cuda-12.0\TensorRT-8.6.1.6\python
    pip install .\tensorrt-8.6.1-cp38-none-win_amd64.whl
    ```

## tensorrt基础
[tensorrt-github](https://github.com/NVIDIA/TensorRT)
+ 在部署阶段，latency是非常重要的点，而TensorRT是专门针对部署端进行优化的
+ tensorrt可以优化推理的吞吐量，但是可能未必能够优化延迟。比如一个推理使用tensorflow可能需要10ms，然后tensorrt可能只能优化一点到8ms。但是一秒能够处理的数据量却可以有很大的优化。
+ TensorRT去做推断（Inference）的时候是不再需要框架的，Caffe和TensorFlow可以通过Parser来导入，一开始就不需要安装这个框架，给一个Caffe或TensorFlow模型，完全可以在TensorRT高效的跑起来。


### 工作流程
1. 首先输入是一个预先训练好的FP32的模型和网络，tensorrt需要解析输入的模型和网络。
2. 将模型通过parser等方式输入到TensorRT中，TensorRT可以生成一个Serialization，也就是说将输入串流到内存或文件中，形成一个优化好的engine。此过程将优化模型或者网络。
3. 执行的时候可以调取engine来执行推断（Inference）。从文件或者内存中反序列化得到模型或者网络。

### TensorRT所做的优化
1. 最重要的，它把一些网络层进行了合并。尽可能并行。
2. 删除一些非必要的层。比如在concat这一层，比如说这边计算出来一个1×3×24×24，另一边计算出来1×5×24×24，concat到一起，变成一个1×8×24×24的矩阵，这个叫concat这层这其实是完全没有必要的，因为TensorRT完全可以实现直接接到需要的地方，不用专门做concat的操作，所以这一层也可以取消掉。
3. Kernel可以根据不同的batch size 大小和问题的复杂程度，去选择最合适的算法，TensorRT预先写了很多GPU实现，有一个自动选择的过程。
4. 不同的batch size会做tuning
5. 不同的硬件如P4卡还是V100卡甚至是嵌入式设备的卡，TensorRT都会做优化，得到优化后的engine。

### ONNX模型转Tensorrt
可以使用tensorrt自带的工具，将onnx的模型转换成tensorrt。比如：
```
// windows下
trtexec.exe --onnx=wav2lip_dec.onnx --minShapes=audio_embed:8x512x1x1,img:8x3x256x256 --optShapes=audio_embed:8x512x1x1,img:8x3x256x256 --maxShapes=audio_embed:8x512x1x1,img:8x3x256x256 --saveEngine=dec.trt --workspace=1024
```

### 经验汇总
+ 生成模trt模型文件的工具版本（trtexec的版本）需要与部署的推理运行环境的tensorrt版本保持一致，否则运行推理时加载模型文件将失败。
    > [TRT] [E] 6: The engine plan file is not compatible with this version of TensorRT, expecting library version 8.6.1.6 got 8.6.0.12, please rebuild.
+ 

### 参考资料
[高性能深度学习支持引擎实战——TensorRT](https://zhuanlan.zhihu.com/p/35657027)

___
总结一下推断（Inference）和训练（Training）的不同：

1. 推断（Inference）的网络权值已经固定下来，无后向传播过程，因此可以

1）模型固定，可以对计算图进行优化

2) 输入输出大小固定，可以做memory优化（注意：有一个概念是fine-tuning，即训练好的模型继续调优，只是在已有的模型做小的改动，本质上仍然是训练（Training）的过程，TensorRT没有fine-tuning

2. 推断（Inference）的batch size要小很多，仍然是latency的问题，因为如果batch size很大，吞吐可以达到很大，比如每秒可以处理1024个batch，500毫秒处理完，吞吐可以达到2048，可以很好地利用GPU；但是推断（Inference）不能做500毫秒处理，可以是8或者16，吞吐降低，没有办法很好地利用GPU.

3. 推断（Inference）可以使用低精度的技术，训练的时候因为要保证前后向传播，每次梯度的更新是很微小的，这个时候需要相对较高的精度，一般来说需要float型，如FP32，32位的浮点型来处理数据，但是在推断（Inference）的时候，对精度的要求没有那么高，很多研究表明可以用低精度，如半长（16）的float型，即FP16，也可以用8位的整型（INT8）来做推断（Inference），研究结果表明没有特别大的精度损失，尤其对CNN。更有甚者，对Binary（二进制）的使用也处在研究过程中，即权值只有0和1。目前FP16和INT8的研究使用相对来说比较成熟。低精度计算的好处是一方面可以减少计算量，原来计算32位的单元处理FP16的时候，理论上可以达到两倍的速度，处理INT8的时候理论上可以达到四倍的速度。当然会引入一些其他额外的操作，后面的讲解中会详细介绍FP18和INT8；另一方面是模型需要的空间减少，不管是权值的存储还是中间值的存储，应用更低的精度，模型大小会相应减小。