# MNN
## MNN库编译、安装
### windows
[官方文档](https://mnn-docs.readthedocs.io/en/latest/compile/engine.html#windows)
+ 进入命令行cmd窗口时应该由系统开始菜单下的visual studio 2019展开，然后点击Developer promote for VS 2019，这样打开的cmd窗口才是设置了X64编译环境。
+ 然后再参照官方文档
```
powershell # 运行该命令从cmd环境进入powershell环境，后者功能更强大
./schema/generate.ps1
# CPU, 64位编译
.\package_scripts\win\build_lib.ps1 -path MNN-CPU/lib/x64
# CPU, 32位编译
.\package_scripts\win\build_lib.ps1 -path MNN-CPU/lib/x86
# CPU+OpenCL+Vulkan, 64位编译
.\package_scripts\win\build_lib.ps1 -path MNN-CPU-OPENCL/lib/x64 -backends "opencl,vulkan"
# CPU+OpenCL+Vulkan, 32位编译
.\package_scripts\win\build_lib.ps1 -path MNN-CPU-OPENCL/lib/x86 -backends "opencl,vulkan"
```
+ powershell可能需要获取执行ps1脚本的策略权限。[参考](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.3)
```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```


### linux编译
```
#安装opengl的依赖库
sudo apt install libglew2.2 libglew-dev

cmake -S . -B build -DCMAKE_BUILD_TYPE:String=Debug -DMNN_BUILD_DEMO=ON -DMNN_BUILD_TOOLS=ON -DMNN_BUILD_CONVERTER=ON -DMNN_OPENCL=ON -DMNN_OPENGL=ON -DMNN_BUILD_TEST=ON -DCMAKE_INSTALL_PREFIX=./output-linux

# 在linux下如果带上MNN_OPENGL=ON则可能连接时报错 -lGLESv3 找不到，不想折腾就把MNN_OPENGL关闭
cmake -S . -B build -DCMAKE_BUILD_TYPE:String=Debug -DMNN_BUILD_DEMO=ON -DMNN_BUILD_TOOLS=ON -DMNN_BUILD_CONVERTER=ON -DMNN_OPENCL=ON -DMNN_BUILD_TEST=ON -DCMAKE_INSTALL_PREFIX=./output-linux

```
注意：
==不要使用-DMNN_TENSORRT=ON==，因为代码中使用的tensorrt版本很低。如果系统中安装的是较新版本的tensorrt，应该无法编译通过。
==-DMNN_SUPPORT_RENDER=ON暂时也还无法编译通过。==
== -DMNN_CUDA=ON编译选项也有问题，因为source/backend/cuda/CMakeLists.txt中没有将cpp加入编译，导致编译出来的so无法链接==
+ 默认情况下是编译生成shared so，此时MNN_SEP_BUILD=ON，会生成libMNN_CL.so、libMNN_GL.so等特定backend的so。
    > cp build-thinkbook/source/backend/opencl/libMNN_CL.so ../Ultra-Light-Fast-Generic-Face-Detector-1MB/MNN/mnn/lib

## 源码分析
| 模块 | 文件 | 功能、内容 | 重要类 |
| -- | -- | -- | -- |
| 网络结构 | schema/current/MNN_generated.h | MNN文件内结构体以及如何表达网络结构的相关类 | struct MNN::Net |
| OpenCL | source/backend/opencl/core/OpenCLBackend.cpp | opencl backend。注册Runtime的Creator。使用CLRuntime类封装OpenCLRuntime类。 | OpenCLBackend、CLRuntime |

## MNN模型转换
```shell
pip install mnn
export PATH=/home/xmagic/anaconda3/envs/tts/bin/:$PATH
mnnconvert -f ONNX --modelFile kokoro-v1.1-zh.onnx --fp16 --MNNModel kokoro-v1.1-zh.mnn --bizCode biz
```