## onnx知识备忘
### ONNX（Open Neural Network Exchange）
微软和Facebook提出用来表示深度学习模型的开放格式。所谓开放就是ONNX定义了一组和环境，平台均无关的标准格式，来增强各种AI模型的可交互性。

### 下载与安装onnx runtime
[官网指引-首页选平台与语言](https://onnxruntime.ai/)
+ c++的dll也需要通过nuget下载，实在不好用。也可以直接从nuget的网站上下载包。[下载地址](https://www.nuget.org/packages/Microsoft.ML.OnnxRuntime.gpu)。点击Download package可以下载到文件microsoft.ml.onnxruntime.gpu.xxx.nupkg。将此文件改为zip后缀即可解压。解压后里面会包含C++所需的头文件以及dll和so。

### 模型内容可视化
#### NetDrawer
参考这个工具的使用教程。[net drawer tool](https://github.com/onnx/tutorials/blob/main/tutorials/VisualizingAModel.md)
1. 安装graphviz[下载地址](https://graphviz.org/download/)。安装后先用命令行或者控制台确认下dot命令可用，不可用则要手动设置下系统环境变量。
    > windows下graphviz安装后还需要修改一下配置文件。[参考](https://blog.csdn.net/qq_36853469/article/details/103555094)
    > 在python安装环境下找到pydot.py文件，将文件内内容 self.prog = 'dot' 改为 self.prog='dot.exe'
2. 在onnx-github源码中下载onnx/tools/net_drawer.py。可以直接clone整个onnx代码库。
3. 运行net_drawer.py
```powershell
Set-ExecutionPolicy -ExecutionPolicy ByPass -Scope CurrentUser
C:\ProgramData\anaconda3\shell\condabin\conda-hook.ps1
conda activate onnx-learn
python .\net_drawer.py --input D:\project\run_server\模型文件\onnx-未加密-2d\wav2lip_enc_audio.onnx --output F:\code\onnx\enc.dot --embed_docstring
dot -Tsvg F:\code\onnx\enc.dot > F:\code\onnx\enc.svg
// 再用浏览器打开F:\code\onnx\enc.svg文件即可预览
```
#### Netron
这个简单得多，直接[下载并安装](https://github.com/lutzroeder/Netron)，然后选择onnx模型文件即可。

### 模型池
[model_zoo](https://github.com/onnx/models)

### 限制
+ Running a model with inputs. ==These inputs must be in CPU memory, not GPU.==

### 参考资料
[onnx-github](https://github.com/onnx/onnx/tree/main)
[onnx教程-github-官方](https://github.com/onnx/tutorials/tree/main)
[ONNX学习笔记](https://zhuanlan.zhihu.com/p/346511883)

2023-06-01 暂停onnx的学习，因为其不支持GPU显存作为输入，输出应该也是不行。