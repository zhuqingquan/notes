### 网络学习资源

[廖雪峰的python 教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431643484137e38b44e5925440ec8b1e4c70f800b4e2000)\
[python内置functions](https://docs.python.org/3/library/functions.html)

### 其他基础库使用

```python
# base64
import base64
a = base64.b64encode(s)
print base64.b64decode(a)

# int to char
chr(i)
# char to int
ord(c) 
# string to bytes
str_1 = "Join our freelance network"
str_1_encoded = bytes(str_1,'UTF-8')
str_1_encoded = str_1.encode(encoding = 'UTF-8')
# bytes to string
str_1 = str_1_encoded.decode()
# bytes to bytearray
str_1_bytearray = bytearray(str_1_encoded)
# bytearray to string
str_1 = str_1_bytearray.decode() 
# bytes不可修改
# 轮询bytes
for bytes in str_1_encoded:
    print(bytes, end = ' ')
# 轮询bytearray每个元素并修改
size = len(realKey)
for i in range(size):
    realKey[i] = realKey[i] ^ shift
    
decrypedBased64Key = b"AudioDriving2D-00000000-00000008\0\0\0\0"
for pos in range(len(decrypedBased64Key)):
    if(int(decrypedBased64Key[pos])==0):
        decrypedBased64Key = decrypedBased64Key[0:pos]
        break    
# sleep
import time
time.sleep(0.01)
# 进程相关
sys.exit(-1)
os.getpid()
os.getppid()
# 判断pid指向的进程是否存在
pip install psutil
psutil.pid_exists(pid)
```

### 多线程、同步锁、[子进程](https://docs.python.org/zh-cn/3.11/library/subprocess.html)

```python
from threading import Thread
//用新线程执行__InitGeneratorLoadRes方法
init_thread = Thread(target=self.__InitGeneratorLoadRes, args=[opts, initial_model_now])
init_thread.start()

# 线程池
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=5) as t:  # 创建一个最大容纳数量为5的线程池
    task1 = t.submit(spider, 2)  # 通过submit提交执行的函数到线程池中
    print(f"task1: {task1.done()}")  # 通过done来判断线程是否完成
    print(task1.result())  # 通过result来获取返回值

# 锁
from threading import RLock
lock_init = RLock()
lock_init.acquire()
lock_init.release()

#子进程
try:
    #ret = os.system(process.cmd)
    pro = subprocess.Popen(process.cmd, shell=True)
    ret = pro.poll()
    if ret == None:
        app_logger.info(f"start process success and still running. pid={pro.pid} cmd={process.cmd}")
        process.process = pro
    else:
        app_logger.error(f"start process failed or exited. pid={pro.pid} ret={ret} cmd={process.cmd}")
        process.process = None
    #app_logger.info(f'start process {process.cmd} result={ret}')
except Exception as ex:
    app_logger.error(f'start process with exception. cmd={process.cmd} {ex}')
```

### asyncio event_loop 协程
1. event loop 就是一个普通 Python 对象，您可以通过 asyncio.new_event_loop() 创建无数个 event loop 对象
2. 想运用协程，首先要生成一个loop对象，然后loop.run_xxx()就可以运行协程了
3. 有两种方法创建event_loop对象：对于主线程是loop=get_event_loop(). 对于其他线程需要首先loop=new_event_loop(),然后set_event_loop(loop)。
4. 局部的小操作或者只有一个线程的简单测试代码用起来方便一些，但是如果需要使用线程执行各种操作，还是开新线程。特别是在api中需要更加谨慎。
5. 协程中尽量不要使用`time.sleep(t)`，因为这个调用会阻塞整个线程，线程绑定的event loop中运行的所有协程其实都得不到CPU资源，因为相对于这些协程来说，`time.sleep(t)`并没有挂起并让出资源。需要使用`await asyncio.wait(t)`

### 时间相关

    import time
    time.sleep(0.1)         // sleep
    time.time()             // 获取当前时间戳，单位秒，float，可以算出毫秒
    time.localtime()        // 当前的本地时间
    time.asctime()          // 返回时间的格式化字符串
    time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())    // 格式化时间字符串
    // 字符串转时间
    time.mktime(time.asctime(time.localtime(time.time())))

    import datetime
    // 格式化打印当前时间，注意%f为毫秒值
    datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S.%f')

### py文件

后缀名 .py
linux、mac中可以支持双击.py文件允许python程序。需要在文件开始处添加特殊的注释。

    #!/usr/bin/env python3
    print('hello, world')

然后给python文件添加可执行权限。

    $ chmod a+x hello.py

### 布尔类型

True False
and or not 操作类型

### 除法、取整、求余

除法 /  取整 //  求余 %

### 字符串

类型 str\
Python 3版本中，字符串是以Unicode编码的
使用以下语句标示.py文件内容的编码格式：

    # -*- coding: utf-8 -*-

ord('a') //获取字符的整数值\
chr(65) //将整数编码值转换成字符\
使用encode('utf8')进行字符串编码格式直接的转换成bytes数据\
使用decode('UTF-8')将接收到的bytes数据转换成str。\
使用len('')获取字符串中字符个数

#### 字符串格式化

%d 整数  %s 字符串  %f 浮点数  %x 十六进制数\
%% 对%的转义\
replace(x, y) 替换内容，会产生新的字符串

    // int 16进制显示8字节，右对齐，不够补0
    "AudioDriving2D-{:0>8x}-00000000".format(os.getppid())

### list

[初级详细的教程](https://www.geeksforgeeks.org/python-lists/)

*   la = \[3, 2]
*   列表下标支持负数，-1取得最后一个项
*   append()插入数据在末尾
*   insert()在下标位置插入
*   pop()删除最后元素，pop(i)删除下标为i的元素
*   sort()  排序
*   list元素的类型可以不同

#### 生成列表

    // 使用range协助生成
    list(range(1,11))
    // 使用for循环生成
    [x*x for x in range(1,11)]
    // 使用for循环并加入条件生成
    [x*x for x in range(1,11) if x%2 == 0]
    // 使用两层循环
    [m+n for m in 'ABC' for n in 'XYZ']

#### list以下方法是线程安全的

[关于python的线程安全](https://juejin.cn/post/6844903615824396295)
L.append(x)
L1.extend(L2)
x = L\[i]
x = L.pop()
L1\[i\:j] = L2
L.sort()
x = y
x.field = y
D\[x] = y
D1.update(D2)
D.keys()

### tuple元组

*   内容初始化之后就不可修改
*   只有一个元素的tuple定义是需要在元素后面加逗号，否则可能有问题
*   a = tuple(1,)

### dict & set

#### dict

*   dict使用{ }大括号进行初始化

    d = { 'michal' : 65, 'bob' : 78}

*   使用key进行赋值时，如果key不存在，则直接插入。

*   使用\[key]进行查询时，如果key不存在，则直接报错

*   判断key是否存在：  key in dict  返回True or False

*   get(key), get(key, defaultVal) 如果key不存在则返回None或者用户定义的值defaultVal。

*   pop(key)

*   key必须是不可变对象

*   内部使用hash表实现

#### set 无序不重复的元素集合

*   只有key，没有value

*   使用list进行创建、初始化

    s = set(\[1,2,3])

*   add(key) 添加

*   remove(key)  删除

### 条件判断

*   条件语句之后要带有冒号

#### if-elseif-else

*   非0数值、非空字符串、非空列表都为True

    if x:
    elif y:
    else:

#### for-in

    for x in y :
        print(x)

只要是可迭代对象，无论有无下标，都可以迭代\
for key in dict   迭代dict的key\
for value in dict.values()  迭代dict的value\
for k,v in dict.items()  迭代dict的item\
判断一个对象是否可迭代的方法：

    from collections import Iterable
    isinstance('abc', Iterable)

Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身

#### while

    while x :  
        yyy

#### 迭代器 Iterator

可以被next()函数调用并不断返回下一个值的对象称为迭代器
判断迭代器的方式：

    from collections import Iterator
    isinstance( (x for x in range(10)), Iterator )

生成器是迭代器Iterator对象，但list，dict、str虽然是Iterable，但不是迭代器Iterator。\
可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过next()函数实现按需计算下一个数据，所以Iterator的计算是惰性的，只有在需要返回下一个数据时它才会计算。\
把list、dict、str等Iterable变成Iterator可以使用iter()函数

### 类型转换

#### str-->int

a = '123'
b = int(a)

#### str-->float

float('12.4')

#### (int, float) --> str

str(120)  str(1.3)

#### (int, str) --> bool

bool(1) bool('')

### 函数定义

#### 定义函数：

    def func(x) :
        return x

#### 导入库lib中的函数：

    from pyfile import function_name

#### 多个返回值

    def func(x):
        return y,z

    y1,z1 = func(a)
    print(y1, z1)

    t1 = func(a) // t1 is a tuple
    print(t1)

返回值是一个tuple！但是，在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值，所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。

#### 默认参数

有多个默认参数时，调用的时候，既可以按顺序提供默认参数。也可以不按顺序提供部分默认参数。当不按顺序提供部分默认参数时，需要把参数名写上。比如调用enroll('Adam', 'M', city='Tianjin')\
**定义默认参数要牢记一点：默认参数必须指向不变对象！**\
原因解释如下：\
Python函数在定义的时候，默认参数L的值就被计算出来了，即\[]，因为默认参数L也是一个变量，它指向对象\[]，每次调用该函数，如果改变了L的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的\[]了。

#### 可变参数函数

定义

    def func(*args)
        pass
    // python允许在list或者tuple变量前面加入*号作为可变参数调用函数
    nums = [1, 2, 3]
    func(*nums)

在函数内部，args接收到的参数是一个tuple

#### 关键字参数

关键字参数允许调用者传入0到n个含参数名的参数。这些关键字参数在函数内部自动组装成一个dict。

    def person(name, age, **kw)
        print('name:', name, 'age:', age, 'other': kw)

    // call person
    person('Bob', 45, city='Beijing')
    // output:  name: Bob age: 35 other: {'city': 'Beijing'}

可以直接将dict变量作为可变参数传入函数。函数内获得的是关键字参数的一份拷贝。对这个拷贝的修改，不会影响调用参数。

    extra = { 'city': 'beijing', 'job':'Engineer' }
    person('jack', 24, **extra)

#### 命名关键字参数

如果要限制关键字参数的名字，则可以使用命名关键字参数。

    // 命名关键字参数需要一个特殊分隔符*，*后面的参数被视为命名关键字参数
    def person(name, age, *, city, job)
        print(name, age, city, job)
    person('jack', 24, city='Beijing', job='Engineering')
    //如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了
    def person(name, age, *args, city, job)
        print(name, age, args, city, jos)

#### 参数组合顺序问题

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

***

### 高级特性

#### 切片（取部分元素）

可支持对list、tuple、字符串等类型数据进行切片\
L\[n\:m] 取下标为\[n,m)的元素，个数为m-n\
L\[:m] 取下标为\[0,m)的元素，个数为m\
n、m可以为负数，但是m必定大于n\
L\[:] 复制整个列表

#### 生成器 Generator

如果列表元素可以按照某种算法推算出来，那么就可以在循环过程中推算出后续的元素，这样就不需要生成完整的list，节约空间。这种一边循环一边计算的机制，叫做生成器。\
创建生成器的方法一：

    L = (x*x for x in range(1, 11)) // 此处为小括号
    for n in L:             //生成器可迭代
        print(n)

创建生成器的方法二：
如果一个函数定义中包含**yield**关键字，那么这个函数不再是普通函数，而是一个生成器。
此生成器直到没有yield再调用时终止。

    def fib(max):
        n, a, b = 0, 0, 1
        while n < max:
            yield b             //b插入生成器的下一个迭代
            a, b = b, a+b
            n = n + 1
        return 'done'

***

### 函数

*   函数名就是指向函数的变量
*   一个函数可以作为另一个函数的调用参数，这种称为**高阶函数**

#### 几种重要的高阶函数

##### map/reduce

*   map(func, Iterable) 将函数依次作用于序列中的每一个元素，并把新的结果作为迭代器Iterator返回。
*   map作为高阶函数，事实上是把运算规则抽象了。并将计算规则应用于每一个元素。
*   reduce把一个需要2个参数的函数作用于序列。reduce继续把结果与序列的下一个元素做累积运算。

    reduce(f, \[x1, x2, x3]) = f(f(x1, x2), x3)

##### filter 过滤

*   filter把传入的函数依次作用于序列中的每一个元素，函数返回True保留该元素，函数返回False则删除该元素。

##### sorted 排序

*   sorted()函数也是一个高阶函数，可以接受一个key函数来实现自定义的排序。
*   传入参数reverse=True实现反向排序

    sorted(\[36, 5, -12, 9, -21], key=abs)
    // 输出 \[5, 9, -12, -21, 36]

***

### 常用函数

*   range(i) -- 生成整数序列
*   isinstance(x, class\_or\_tuple) -- 判断变量x是否为类型集合中类型的对象实例
*   os.listdir('dir') -- 获取文件夹的文件列表

### 运行时与环境

#### anaconda3

在 Python 开发中，很多时候我们希望每个应用有一个独立的 Python 环境（比如应用 1 需要用到 TensorFlow 1.X，而应用 2 使用 TensorFlow 2.0）。这时，Conda 虚拟环境即可为一个应用创建一套 “隔离” 的 Python 运行环境。使用 Python 的包管理器 conda 即可轻松地创建 Conda 虚拟环境。
下载并安装anaconda3。[下载地址](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M\&O=A)。linux是先下载个sh文件，再执行sh即可以安装。

*   Windows下安装后，启动conda简单的方式是：从开始-->Anaconda prompt启动Anaconda虚拟环境。首次安装pytorch需要管理员身份运行Anaconda

*   创建新的虚拟环境并进入该虚拟环境
    常用命令如下：

    conda create --name \[env-name] python=x.x     # 建立名为\[env-name]的Conda虚拟环境
    conda activate \[env-name]           # 进入名为\[env-name]的Conda虚拟环境
    conda deactivate                    # 退出当前的Conda虚拟环境
    conda env remove --name \[env-name]  # 删除名为\[env-name]的Conda虚拟环境
    conda env list                      # 列出所有Conda虚拟环境
    conda list                          # 查看当前ENV下安装的包的列表
    conda info | grep 'active environment' | grep fastai  # 判断当前ENV是否为fastai

    # 修改env下的python版本

    conda install python=3.10
    conda update python

*   windows安装后需要添加路径到PATH环境变量才能正常在命令行使用conda命令，比如

    C:\ProgramData\anaconda3
    C:\ProgramData\anaconda3\Scripts
    C:\ProgramData\anaconda3\Library\bin

*   vscode下如果需要在terminal中使用conda，还需要运行以下命令

    C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy ByPass -NoExit -Command "& 'C:\Users\zhuqu\anaconda3\shell\condabin\conda-hook.ps1' ;  conda activate audio-driving-2d-onnx "

    // PowerShell更改策略允许当前用户执行ps文件
    Set-ExecutionPolicy -ExecutionPolicy ByPass -Scope CurrentUser
    // 设置conda环境
    C:\ProgramData\anaconda3\shell\condabin\conda-hook.ps1
    // or
    C:\Users\zhuqu\anaconda3\shell\condabin\conda-hook.ps1
    // 之后再调用conda activate才能成功
    conda activate audio-driving-2d-onnx

*   linux下修改.bashrc初始化anaconda

```
    # >>> conda initialize >>>

    # !! Contents within this block are managed by 'conda init' !!

    \_\_conda\_setup="`$('/home/ninja/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $`? -eq 0 ]; then
    eval "`$__conda_setup"
    else
        if [ -f "/home/ninja/anaconda3/etc/profile.d/conda.sh" ]; then
            . "/home/ninja/anaconda3/etc/profile.d/conda.sh"
        else
            export PATH="/home/ninja/anaconda3/bin:$`PATH"
    fi
    fi
    unset \_\_conda\_setup

    # <<< conda initialize <<<
```
*   设置conda启动时不直接activate

    ```conda config --set auto\_activate\_base false```

#### 将conda env迁移到另一台机

    ## You can try below approach to move all the package from one machine to other :
    ## Note : Machine that packages are being moved should be same and python version also should be same

    $ pip install conda-pack

    # To package an environment:
    ## Pack environment my_env into my_env.tar.gz
    $ conda pack -n my_env

    # After following above approach, you will end up with a tar.gz file. Now to install package from this zip file follow below approach.

    ## To install the environment:

    ## Unpack environment into directory `my_env`

    $ mkdir -p my_env
    $ tar -xzf my_env.tar.gz -C my_env


    ## Use Python without activating or fixing the prefixes. Most Python
    ## libraries will work fine, but things that require prefix cleanups
    ## will fail.

    $ ./my_env/bin/python


    ## Activate the environment. This adds `my_env/bin` to your path

    $ source my_env/bin/activate


    ## Run Python from in the environment

    (my_env) $ python


    ## Cleanup prefixes from in the active environment.
    ## Note that this command can also be run without activating the environment
    ## as long as some version of Python is already installed on the machine.
    (my_env) $ conda-unpack

#### pip

更新pip的源为清华的
python -m pip install -i <https://pypi.tuna.tsinghua.edu.cn/simple> --upgrade pip
pip config set global.index-url <https://pypi.tuna.tsinghua.edu.cn/simple>

#### 其他国内镜像源

清华大学TUNA镜像源： <https://pypi.tuna.tsinghua.edu.cn/simple>
阿里云镜像源： <https://mirrors.aliyun.com/pypi/simple/>
中国科学技术大学镜像源： <https://mirrors.ustc.edu.cn/pypi/simple/>
华为云镜像源： <https://repo.huaweicloud.com/repository/pypi/simple/>
腾讯云镜像源：<https://mirrors.cloud.tencent.com/pypi/simple/>

#### vs code + python

1.  vscode将默认安装python插件
2.  在vscode使用python: selecte interpreter可以选择虚拟环境。anaconda3中创建的环境也可以选择。

#### 调试python代码内存泄漏

工具： [memory-profiler](https://zhuanlan.zhihu.com/p/645260012)、[filpprofiler](https://zhuanlan.zhihu.com/p/397643830)
[memory-profiler官网-用法演示](https://pypi.org/project/memory-profiler/)

*   filpprofiler可能更好用，但是只支持linux与macos
*   memory-profiler只能排查python对象或者申请的内存的泄漏，而如果是调用系统api导致的泄漏，则通过memory-profiler是检测不到的。

## nuitka3打包工具安装

```shell
pip install nuitka==1.5.8 -i https://pypi.tuna.tsinghua.edu.cn/simple
# Windows系统下可能不需要执行以下命令。nuitka支持在linux下打包Windows可执行程序，此时需要安装比如patchelf等
conda install python==3.9.16
conda install libpython-static==3.9.16
sudo apt install patchelf
```

#### 将python文件打包成可执行文件（Windows）

```shell
# 安装nuitka
pip install nuitka==1.5.8 -i https://pypi.tuna.tsinghua.edu.cn/simple
# 对包含main()的python进行打包
nuitka --standalone --output-dir=dist service_daemon.py
# 若需要支持cuda
nuitka --standalone --include-package=cuda --include-module=cuda._lib.utils  --include-module=cuda.ccudart --include-module=cuda.ccuda --include-module=cuda._cuda.ccuda --include-module=cuda._lib.ccudart.utils --include-module=cuda._lib.ccudart.ccudart --output-dir=out run_server.py
# linux下使用这个格式执行
python -m nuitka --standalone --output-dir=dist service_daemon.py
```

***

## python的快

### 快速部署http api服务器

#### 使用FastAPI库

[官方教程](https://fastapi.tiangolo.com/tutorial/)

1.  安装fastapi+uvicorn

    pip install fastapi
    pip install "uvicorn\[standard]"

2.  代码示例

```python
import asyncio
from fastapi import FastAPI
from fastapi.logger import logger as fastapi_logger
import uvicorn

faceDetectSvc = FastAPI()

def image_pretest_func(app_id, uid, request_id, image_type, image_base64):
    return {}

@faceDetectSvc.post('/facedetect_api/face_detect')
async def image_pretest_service(info: ImageInfo):
    loop = asyncio.get_event_loop()
    res = await loop.run_in_executor(executor_pretest, image_pretest_func, app_id, uid, request_id, info.image_type, image_base64)
    return res
    
if __name__ == '__main__':
    uvicorn.run(faceDetectSvc, host='0.0.0.0', port=11030, reload=False)
```

1.

***

## 其他

### 编译proto文件的python代码

```shell
pip install grpcio
pip install grpcio-tools
python -m grpc_tools.protoc -I./protos --python_out=./protos --pyi_out=./protos --grpc_python_out=./protos .\protos\audio_driving.proto
```

### opencv库的使用代码片段

```python
import cv2
import numpy as np

# 将jpeg或者png等数据解码成rgb，jpeg或者npg数据可以从文件中读取或者网络传输
nparr = np.frombuffer(image, np.uint8)
img_np = cv2.imdecode(nparr, cv2.IMREAD_COLOR) # cv2.IMREAD_COLOR in OpenCV 3.1
# 将rgb数据显示在窗口中预览或者写入jpeg文件
cv2.imshow("FaceDetectSvc preview", img_np)
cv2.imwrite("result.jpg", img_np) 
```

