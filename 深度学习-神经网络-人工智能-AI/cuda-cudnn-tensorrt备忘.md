## 文档
[developer-guide](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#build-phase)

## 模型优化
The builder eliminates dead computations, 
folds constants, 
and reorders and combines operations to run more efficiently on the GPU. 
It can optionally reduce the precision of floating-point computations, either by simply running them in 16-bit floating point, or by quantizing floating point values so that calculations can be performed using 8-bit integers.
It also times multiple implementations of each layer with varying data formats, then computes an optimal schedule to execute the model, minimizing the combined cost of kernel executions and format transforms.

## Windows安装CUDA

[从官网下载安装包并安装](https://developer.nvidia.com/cuda-downloads?target_os=Windows\&target_arch=x86_64\&target_version=11\&target_type=exe_local)

*   [安装NVIDIA Nsight Integration for vs stdio](https://developer.nvidia.com/nvidia-nsight-integration-install-tips)
*   [安装NVIDIA Nsight Graphics](https://developer.nvidia.com/nsight-graphics)
    NVIDIA Nsight Graphics用于调试GPU问题，也可以调试OpenGL的问题。
*   [NVIDIA Nsight Integration](https://developer.nvidia.com/nsight-tools-visual-studio-integration)
*   安装后确认是否成功以及查看版本号

<!---->

    nvcc --version

### windows下WSL内安装CUDA

1.  确认Windows系统下的NVIDIA驱动是支持WSL的版本，一般最新版本的驱动都支持WSL。
2.  [按照官网的指引安装特定版本CUDA](https://developer.nvidia.com/cuda-downloads?target_os=Linux\&target_arch=x86_64\&Distribution=WSL-Ubuntu\&target_version=2.0\&target_type=deb_local)
3.  如果已下载了deb，则可参考并直接安装

<!---->

    wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
    sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
    wget https://developer.download.nvidia.com/compute/cuda/12.1.1/local_installers/cuda-repo-wsl-ubuntu-12-1-local_12.1.1-1_amd64.deb
    sudo dpkg -i cuda-repo-wsl-ubuntu-12-1-local_12.1.1-1_amd64.deb
    sudo cp /var/cuda-repo-wsl-ubuntu-12-1-local/cuda-*-keyring.gpg /usr/share/keyrings/
    sudo apt-get update
    sudo apt-get -y install cuda

1.  ~~安装nvidia-cuda-toolkit~~
    如果使用cuda的库安装了，则不用再安装此toolkit，因为/usr/local/cuda/bin中包含了这些toolkit。

<!---->

    sudo apt install nvidia-cuda-toolkit

1.  设置环境变量
    在\~/.bashrc后面添加以下内容

<!---->

    export CUDA_HOME=/usr/local/cuda
    export PATH=$PATH:$CUDA_HOME/bin
    export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

1.  source \~/.bashrc

### linux下安装

基本与windows下WSL内的安装一致。
下载的deb包不一样

    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
    sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
    wget https://developer.download.nvidia.com/compute/cuda/12.1.1/local_installers/cuda-repo-ubuntu2204-12-1-local_12.1.1-530.30.02-1_amd64.deb
    sudo dpkg -i cuda-repo-ubuntu2204-12-1-local_12.1.1-530.30.02-1_amd64.deb
    sudo cp /var/cuda-repo-ubuntu2204-12-1-local/cuda-*-keyring.gpg /usr/share/keyrings/
    sudo apt-get update
    sudo apt-get -y install cuda

### ==ubuntu 20.04下安装cuda 11.8==
正常安装失败了。
1. 正常安装完ubuntu 20.04时启动进入的是内核5.16。这个内核版本与TT服务器使用的5.4版本的内核不同。需要启动时手动切换一下内核版本。
2. 进入ubuntu 20.04（不管哪个内核），使用sudo apt isntall nvidia-driver-520安装520版本的nvidia驱动总是失败。原因是安装是nvidia-dkms-520只支持GPL授权，此时无法安装。
3. 如果直接安装nvidia 535驱动，则此时安装cuda 11.8的deb包时，deb包会把535版本的驱动卸载掉然后自动安装其deb内部的nvidia 520驱动。此时又会安装驱动失败，进而导致安装cuda失败。

解决方案1：
安装nvidia 535驱动，然后再使用cuda 11.8的run包进行安装，不要使用deb。
```
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
sudo sh cuda_11.8.0_520.61.05_linux.run
```

## 安装cudnn（Windows）

[下载地址](https://developer.nvidia.com/rdp/cudnn-download)    ==注意需要下载与cuda对应版本的cudnn。==
[官方安装指引](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)

*   注意：安装官方指引是将cudnn的include、lib、bin单独放在一个文件夹里面。但是其实将他们放在cuda的目录下对应的include、lib、bin下面会更方便一点。
    *   include拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\include
    *   lib拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\lib\x64
    *   bin拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin

## 安装cudnn（linux & wsl）

1.  [下载zip包，不用deb](https://developer.nvidia.com/rdp/cudnn-download)
2.  下载完后解压压缩包

<!---->

    tar -xvf d/cudnn-linux-x86_64-8.9.2.26_cuda12-archive.tar.xz -C ./cudnn

1.  拷贝头文件与lib到cuda目录下

<!---->

    sudo cp cudnn/cudnn-linux-x86_64-8.9.2.26_cuda12-archive/include/* /usr/local/cuda/include/
    sudo cp cudnn/cudnn-linux-x86_64-8.9.2.26_cuda12-archive/lib/* /usr/local/cuda/lib64/
    sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
1.

9.2.0版本的既可以支持cuda-11，也可以安装支持cuda-12
```
wget https://developer.download.nvidia.com/compute/cudnn/9.2.0/local_installers/cudnn-local-repo-ubuntu2204-9.2.0_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2204-9.2.0_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2204-9.2.0/cudnn-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
#sudo apt-get -y install cudnn
#To install for CUDA 11, perform the above configuration but install the CUDA 11 specific package:
sudo apt-get -y install cudnn-cuda-11
#To install for CUDA 12, perform the above configuration but install the CUDA 12 specific package:
sudo apt-get -y install cudnn-cuda-12
```


## 安装tensorrt(Windows)

*   GA与EA的区别
    EA 版本代表抢先体验（在正式发布之前）。
    GA 代表通用性。 表示稳定版，经过全面测试。
    [下载页面](https://developer.nvidia.com/nvidia-tensorrt-download)
    [下载页面-8x](https://developer.nvidia.com/nvidia-tensorrt-8x-download)==注意下载与cuda对应版本的tensorrt==
*   直接解压拷贝include、lib、bin到cuda安装目录下
    *   include拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\include
    *   lib拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\lib\x64
    *   bin拷贝到--> C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin
*   [可以参考](https://zhuanlan.zhihu.com/p/339753895)
*   以上安装的只是c/c++的tensorrt库。
*   python库的安装
    *   下载的tensorrt zip包中也包含python的whl安装文件。可以直接还用pip install安装

    <!---->

        cd TensorRT-8.6.1.6.Windows10.x86_64.cuda-12.0\TensorRT-8.6.1.6\python
        pip install .\tensorrt-8.6.1-cp38-none-win_amd64.whl
        pip install .\tensorrt_dispatch-8.6.1-cp38-none-win_amd64.whl
        pip install .\tensorrt_lean-8.6.1-cp38-none-win_amd64.whl

## tensorrt基础

*   在部署阶段，latency是非常重要的点，而TensorRT是专门针对部署端进行优化的
*   tensorrt可以优化推理的吞吐量，但是可能未必能够优化延迟。比如一个推理使用tensorflow可能需要10ms，然后tensorrt可能只能优化一点到8ms。但是一秒能够处理的数据量却可以有很大的优化。

### 工作流程

1.  首先输入是一个预先训练好的FP32的模型和网络，tensorrt需要解析输入的模型和网络。
2.  将模型通过parser等方式输入到TensorRT中，TensorRT可以生成一个Serialization，也就是说将输入串流到内存或文件中，形成一个优化好的engine。此过程将优化模型或者网络。
3.  执行的时候可以调取engine来执行推断（Inference）。从文件或者内存中反序列化得到模型或者网络。

### ONNX模型转Tensorrt

可以使用tensorrt自带的工具，将onnx的模型转换成tensorrt。比如：

    // windows下
    trtexec.exe --onnx=wav2lip_dec.onnx --minShapes=audio_embed:8x512x1x1,img:8x3x256x256 --optShapes=audio_embed:8x512x1x1,img:8x3x256x256 --maxShapes=audio_embed:8x512x1x1,img:8x3x256x256 --saveEngine=dec.trt --workspace=1024

### 经验汇总

*   生成模trt模型文件的工具版本（trtexec的版本）需要与部署的推理运行环境的tensorrt版本保持一致，否则运行推理时加载模型文件将失败。
    > \[TRT] \[E] 6: The engine plan file is not compatible with this version of TensorRT, expecting library version 8.6.1.6 got 8.6.0.12, please rebuild.
*

### 参考资料

[高性能深度学习支持引擎实战——TensorRT](https://zhuanlan.zhihu.com/p/35657027)

***

## 部署运行

*   运行时指定CUDA程序可见（可使用）的显卡

<!---->

    # 在运行的程序命令行前面加 CUDA_VISIBLE_DEVICES=0,2,3
    CUDA_VISIBLE_DEVICES=3 nohup ./run_server.bin
    # 临时设置：
    Linux： export CUDA_VISIBLE_DEVICES=1
    windows:  set CUDA_VISIBLE_DEVICES=1
    # 永久设置：
    linux:
    在~/.bashrc 的最后加上export CUDA_VISIBLE_DEVICES=1，然后source ~/.bashrc
    windows:
    打开我的电脑环境变量设置的地方，直接添加就行了。

*

***

## CUDA基础
### 资料
[CUDA Programming Guide in Chinese](https://github.com/HeKun-NVIDIA/CUDA-Programming-Guide-in-Chinese)

### 代码编译以及项目管理

*   cuda的代码可以保存在以.cu为后缀的文件中，比如vectorAdd.cu
    编译方式：

1.  直接使用nvcc编译单个文件，感觉和GCC类似

<!---->

    nvcc vectorAdd.cu -o vectorAdd

2.  使用cmake--find\_package

```CMakeLists.txt
cmake_minimum_required(VERSION 3.8)
project(CUDA_EXAMPLES)          
                                
find_package(CUDA REQUIRED)     
                                
message(STATUS "cuda version: " ${CUDA_VERSION_STRING})
message(STATUS "cuda info include_dir: " ${CUDA_INCLUDE_DIRS})
message(STATUS "cuda info libraries: " ${CUDA_LIBRARIES})
                                
include_directories(${CUDA_INCLUDE_DIRS})             
                                
cuda_add_executable(vectorAdd vectorAdd.cu)
target_link_libraries(vectorAdd ${CUDA_LIBRARIES})             
```
find_package成功后，${CUDA_TOOLKIT_ROOT_DIR}就是CUDA库根目录。

上面这个CMakeList.txt可以直接使用 *cmake -S . -B build* 进行config，并用*cmake --build build --config Debug --target all -j 12 --* 进行编译。
3\. 使用cmake --enable\_language(CUDA)

3. cmake config时指定CUDA_TOOLKIT_ROOT_DIR变量的值
cmake -S . -B build -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-12.1  #注意要指定到具体版本
找到CUDA后，可用的变量
CUDA_CUDA_LIBRARY == /usr/lib/wsl/lib/libcuda.so
需要连接libcudart_static.a则指定路径：${CUDA_TOOLKIT_ROOT_DIR}/targets/x86_64-linux/lib

### 线程与Context
1. 首次调用cuda的API时会自动创建与设备相关的cuda context。比如调用cuDeviceSet。
2. CUDA 上下文类似于 CPU 进程。驱动 API 中执行的所有资源和操作都封装在 CUDA 上下文中，当上下文被销毁时，系统会自动清理这些资源。
3. 主机线程一次可能只有一个设备上下文当前。当使用 cuCtxCreate() 创建上下文时，它对调用主机线程是当前的。如果有效上下文不是线程当前的，则在上下文中操作的 CUDA 函数（大多数不涉及设备枚举或上下文管理的函数）将返回 CUDA_ERROR_INVALID_CONTEXT。
4. 每个主机线程都有一堆当前上下文。 cuCtxCreate() 将新上下文推送到堆栈顶部。可以调用 cuCtxPopCurrent() 将上下文与主机线程分离。然后上下文是“浮动的”，并且可以作为任何主机线程的当前上下文推送。 cuCtxPopCurrent() 还会恢复先前的当前上下文（如果有）。
5. 默认情况下，一个进程中，在初次调用CUDA runtime软件库中的任何一个API时，会自动初始化当前进程中唯一的一个CUDA上下文
6. 在没有显式调用cuCtxCreate的情况下，使用cudaMalloc申请的显存可以在其他线程进行操作。
7. 由于GPU无法同时执行跨CUDA context的任务，导致硬件利用率可能不高，此时可采用一些虚拟化手段。典型的如NVIDIA官方的MPS（Multi Process Service），它实际上启动了一个独立进程去转发所有的任务。采用此方法的坏处是隔离性收到了一定破坏：一旦此进程失效，所有关联任务都将受到影响。

### 将cu编译成动态库的cmake文件例子
```CMakeLists.txt
cmake_minimum_required(VERSION 3.18)
project(CudaSharedLibrary LANGUAGES CXX CUDA)

# Specify the CUDA version
set(CMAKE_CUDA_STANDARD 11)

# Add the CUDA source file
add_library(example SHARED example.cu)

# Set the target properties
set_target_properties(example PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
    POSITION_INDEPENDENT_CODE ON
)
```

### 编译错误汇总
1. 如果使用gcc、g++版本大于11，直接编译cu文件会报错，此时可以添加以下环境变量规避
```
export NVCC_APPEND_FLAGS='-allow-unsupported-compiler'
```
或者在cmake文件中添加以下compile_option
```
target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:-allow-unsupported-compiler>)
```

2. 对fp16的编译支持
错误提示：error: identifier "__hadd2" is undefined
解决方式：
    1. 在命令行中导出 export NVCC_APPEND_FLAGS='--gpu-architecture=compute_70'

***

总结一下推断（Inference）和训练（Training）的不同：

1.  推断（Inference）的网络权值已经固定下来，无后向传播过程，因此可以
    (1) 模型固定，可以对计算图进行优化
    (2) 输入输出大小固定，可以做memory优化（注意：有一个概念是fine-tuning，即训练好的模型继续调优，只是在已有的模型做小的改动，本质上仍然是训练（Training）的过程，TensorRT没有fine-tuning

2.  推断（Inference）的batch size要小很多，仍然是latency的问题，因为如果batch size很大，吞吐可以达到很大，比如每秒可以处理1024个batch，500毫秒处理完，吞吐可以达到2048，可以很好地利用GPU；但是推断（Inference）不能做500毫秒处理，可以是8或者16，吞吐降低，没有办法很好地利用GPU.

3.  推断（Inference）可以使用低精度的技术，训练的时候因为要保证前后向传播，每次梯度的更新是很微小的，这个时候需要相对较高的精度，一般来说需要float型，如FP32，32位的浮点型来处理数据，但是在推断（Inference）的时候，对精度的要求没有那么高，很多研究表明可以用低精度，如半长（16）的float型，即FP16，也可以用8位的整型（INT8）来做推断（Inference），研究结果表明没有特别大的精度损失，尤其对CNN。更有甚者，对Binary（二进制）的使用也处在研究过程中，即权值只有0和1。目前FP16和INT8的研究使用相对来说比较成熟。低精度计算的好处是一方面可以减少计算量，原来计算32位的单元处理FP16的时候，理论上可以达到两倍的速度，处理INT8的时候理论上可以达到四倍的速度。当然会引入一些其他额外的操作，后面的讲解中会详细介绍FP18和INT8；另一方面是模型需要的空间减少，不管是权值的存储还是中间值的存储，应用更低的精度，模型大小会相应减小。ffp

4.

***

临时信息：
使用visual studio 2022打开cuda\_samples中的解决方案。
编译vectorAdd项目，产生的编译指令是：

    F:\code\cuda-samples\Samples\0_Introduction\vectorAdd>"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\bin\nvcc.exe" -gencode=arch=compute_50,code=\"sm_50,compute_50\" -gencode=arch=compute_52,code=\"sm_52,compute_52\" -gencode=arch=compute_60,code=\"sm_60,compute_60\" -gencode=arch=compute_61,code=\"sm_61,compute_61\" -gencode=arch=compute_70,code=\"sm_70,compute_70\" -gencode=arch=compute_75,code=\"sm_75,compute_75\" -gencode=arch=compute_80,code=\"sm_80,compute_80\" -gencode=arch=compute_86,code=\"sm_86,compute_86\" -gencode=arch=compute_89,code=\"sm_89,compute_89\" -gencode=arch=compute_90,code=\"sm_90,compute_90\" --use-local-env -ccbin "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\HostX64\x64" -x cu   -I./ -I../../../Common -I./ -I"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\/include" -I../../../Common -I"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1\include"  -G   --keep-dir x64\Debug  -maxrregcount=0   --machine 64 --compile -cudart static -Xcompiler "/wd 4819"  --threads 0 -g  -DWIN32 -DWIN32 -D_MBCS -D_MBCS -Xcompiler "/EHsc /W3 /nologo /Od /FS /Zi /RTC1 /MTd " -Xcompiler "/Fdx64/Debug/vc143.pdb" -o F:\code\cuda-samples\Samples\0_Introduction\vectorAdd\x64\Debug\vectorAdd.cu.obj "F:\code\cuda-samples\Samples\0_Introduction\vectorAdd\vectorAdd.cu"
    vectorAdd.cu

