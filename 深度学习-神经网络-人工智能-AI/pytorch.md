<!--
 * @Author: zhuqingquan zqq_222@163.com
 * @Date: 2025-07-08 12:10:55
 * @FilePath: /notes/深度学习-神经网络-人工智能-AI/pytorch.md
-->
## 源码编译安装
### C++代码的编译结构
| cmake文件 | 项目 | 库 | 说明 |
| ---- | ---- | ---- | ---- |
|pytorch/CMakeLists.txt | Torch | - | 整个pytorch C++ C模块的入口 | 
|pytorch/cmake/public/utils.cmake | - | - | 宏定义与工具函数 |
|pytorch/cmake/Summary.cmake | - | - | 打印所有cmake option的信息 |
|pytorch/cmake/Dependencies.cmake | - | - | 根据option开关配置的USE的库来添加编译参数，或者使用add_subdirectory配置构建依赖库 |
| ==pytorch/c10/CMakeLists.txt== | | | 主要工程代码 |
|==pytorch/caffe2/CMakeLists.txt== | caffe2 | | 核心Caffe2 |
|pytorch/c10/cuda/CMakeLists.txt| c10_cuda | libc10_cuda.so | CUDA运行时封装 |

### 子模块C10
[PyTorch C10 CUDA 模块源码结构解读](https://blog.csdn.net/m0_62405272/article/details/130124736)
[Pytorch开发笔记01：C10 的软件架构](https://zhuanlan.zhihu.com/p/711688484)

### 编译
```shell
# 编译flash-attention很耗时，先关闭
USE_FLASH_ATTENTION=OFF DEBUG=1 MAX_JOBS=8 python setup.py develop
```

#### 只编译C++部分
```
cmake -S . -B build_cpp -DCMAKE_BUILD_TYPE:String=Debug -DUSE_FLASH_ATTENTION=OFF
```