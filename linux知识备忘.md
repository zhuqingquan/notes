### ==常用命令行==

```shell
# 解压
tar -xvf path/file.tar.zip -C dstPath
unzip file.zip 
# 压缩：
#1. 先创建tar文件
tar -cvf dstFile.tar ./sourceDirOrFile
# 2. 再用gzip压缩
gzip dstFile.tar
# 上面2条命令合并，加了-z选项
tar -czvf  5052-20240206.tar.gz logs/libmagic_5052_20240206_105427.*
# 使用zip压缩 -r为递归延时文件夹内容
zip -r file.zip dir   
grep 703[1234]
# 创建特定版本库的软链接
ln -s liba.s0.1.xxx liba.so
# 清理一天前未被访问的资源
find 路径 -type f -atime +1 -print0 | xargs -0 -n 100 rm -f
```

<https://forum.ubuntu.com.cn/viewtopic.php?t=493123>

<https://zhuanlan.zhihu.com/p/601657639>

#### 文件传输SCP

本地文件发送到远程主机上：`scp -P 22 -r src_local username@remote_host:path`
读取远程主机上文件到本地：`scp -P 22 -r username@remote_host:path dst_local`

#### 查看或者更新libc版本

    ldd --version  # 查看版本
    # 以下方式可以将libc更新到与当前kernel版本匹配的最高版本的libc
    sudo apt update
    sudo apt install libc-bin

[在同一台Linux机器上安装多个版本GLIBC](http://fancyerii.github.io/2024/03/07/multi-glibc-patchelf/)

#### c++编程环境

[多个gcc/glibc版本的共存及指定gcc版本的编译](https://blog.csdn.net/mo4776/article/details/119837501)

```shell
# gcc编译
tar -xvf gcc-12.3.0.tar.gz
cd gcc-12.3.0
contrib/download_prerequisites
mkdir ../gc-12.3.0-build
cd ../gc-12.3.0-build
../gc-12.3.0/configure --disable-multilib --host=x86_64-pc-linux-gnu
make
```

[使用update-alternatives的方式安装多个版本的gcc并切换](https://zhuanlan.zhihu.com/p/146205444) -- ==这个方法测试有效，而且较为简单==

    sudo apt install g++-12 gcc-12 g++-11 gcc-11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 120 --slave /usr/bin/g++ g++ /usr/bin/g++-12 --slave /usr/bin/gcov gcov /usr/bin/gcov-12

    sudo update-alternatives --config gcc

#### 查找系统中是否安装了特定的依赖库
```shell
ldconfig -p | grep -E "xcb-xfixes|xcb-shape"
```

#### 缺少头文件或者so时使用apt-file进行搜索需要安装的包

    sudo apt install apt-file
    sudo apt-file update
    apt-file search xxx.h

#### grep + 正则表达式

[参考](https://www.myfreax.com/regular-expressions-in-grep/)

    # 匹配7031或者7032、7033、7034
    grep 703[1234]
    # 查找文件内容，用来分析日志很有用
    grep key file

#### CGDB调试指令

| 指令                            | 作用            |
| ----------------------------- | ------------- |
| sudo apt install cgdb         | 安装cgdb        |
| cgdb client                   | 启动调试client    |
| Esc键                          | 进入代码窗口        |
| i                             | 进入调试命令窗口      |
| 按下ESC，+ / -                   | 调整代码窗口与命令窗口大小 |
| o (处于代码窗口时)                   | 打开代码列表        |
| 空格健 (处于代码窗口时)                 | 添加或者取消断点      |
| F5->r F6->c F7->q F8->n F9->s |               |

#### GDB调试指令备忘

| 指令                                                                   | 作用                                    |
| -------------------------------------------------------------------- | ------------------------------------- |
| info breakpoints                                                     | 打印所有断点信息                              |
| delete                                                               | 删除所有断点                                |
| delete breakpoint \[n]                                               | 删除第n个断点     (watch也是通过这个删除)           |
| disable breakpoint \[n]                                              | 禁用第n个断点                               |
| enable breakpoint \[n]                                               | 启用第n个断点                               |
| save breakpoint filename                                             | 将断点信息保存到文件中                           |
| gdb &lt;program&gt; -x breakpoint\_filename                          | 开始调试时加载断点信息文件                         |
| bt                                                                   | 打印线程堆栈信息                              |
| f n                                                                  | 调整到堆栈的第N层                             |
| up / down                                                            | 堆栈向上或者向下一层                            |
| thread apply all bt                                                  | 打印所有线程信息                              |
| info variables                                                       | 查看全局和静态变量                             |
| info locals                                                          | 查看局部变量                                |
| info args                                                            | 查看参数                                  |
| info sharedlibrary                                                   | 打印加载的动态库列表                            |
| display {var,var2}                                                   | 打印多个变量                                |
| x /\[个数]\[打印格式]\[每个元素字节数] \[内存地址]                                    | 打印内存，显示内存数据 x /nfu addr               |
| gdb \<exe> \<pid> <br> gdb -p \<pid> <br> 先gdb 进去后再执行attach \<pid>指令 | 附加到正在运行的进程pid <br> 附加到其他用户启动的进程需要sudo |
| si                                                                   | 步进，单步调试汇编                             |
| set disassemble-next-line on                                         | 自动打印下一行汇编指令                           |
| i r xmm5                                                             | 显示寄存器值                                |
| set print elements unlimit                                           | 设置显示完整的长字符串                           |

a）n：输出单元的个数。
b）f：是输出格式。比如x是以16进制形式输出，o是以8进制形式输出,等等。
c）u：标明一个单元的长度。b是一个byte，h是两个byte（halfword），w是四个byte（word），g是八个byte（giant word）。

#### install software after install system

    sudo add-apt-repository ppa:kisak/kisak-mesa
    sudo apt update
    sudo apt upgrade
    sudo apt install git gcc g++ clang cmake make fcitx vim ninja-build net-tools apt-file cgdb libgl1-mesa-dev libglu1-mesa libglu1-mesa-dev libegl1-mesa libegl1-mesa-dev libegl-dev mesa-utils pkg-config libglvnd0 libva2 libva-dev libbz2-1.0 libbz2-dev  liblzma5 liblzma-dev zlib1g-dev zlib1g libvdpau-dev libvdpau1 libvdpau-va-gl1 libssl-dev  libboost-system-dev libcurl4-openssl-dev  libharfbuzz0b libharfbuzz-dev libxcb-randr0 libxcb-randr0-dev libbrotli libbrotli-dev libasound2 libasound2-dev libass9 libass-dev  libfontconfig1 libfontconfig1-dev

#### 生成SSH key并提交到ssh-agent

查看以及生成SSH key

    // 查看现有的SSH key
    ls -al ~/.ssh
    // 使用ssh-keygen生成SSH KEY
    ssh-keygen -t ed25519 -C "zqq_222@163.com"
    Enter file in which to save the key (/home/zqq/.ssh/id_ed25519): 此处输入保存路径，使用默认直接回车
    Enter passphrase (empty for no passphrase): 输入一个本地密码，不需要则直接回车
    Enter same passphrase again:确认本地密码
    Your identification has been saved in /home/zqq/.ssh/id_ed25519
    Your public key has been saved in /home/zqq/.ssh/id_ed25519.pub
    The key fingerprint is:
    SHA256:559vY7NG0EqSsnsbReZKamk+2h+ixmLhY4Z97Q+MVGE zqq_222@163.com
    The key's randomart image is:
    +--[ED25519 256]--+
    |        E        |
    |       . .       |
    |        .  .o.   |
    |       .. o+o .  |
    |      . Sooooo   |
    |    .. o.* o. .  |
    |   + o..O.=  .   |
    |  . O +*+o.+ .*  |
    |   + =oo+=+.o=o+ |
    +----[SHA256]-----+
    //直接查看私有KEY
    cat ~/.ssh/id_ed25519

将生成的SSH key加入到SSH agent

    exec ssh-agent  bash
    ssh-add ~/.ssh/id_ed25519

#### 安装minidlna

    sudo apt install minidlna
    // 启动、重启（安装后设置了开机启动）
    sudo service minidlna start
    sudo service minidlna restart
    sudo service minidlna stop
    sudo service minidlna status

    // 修改配置
    sudo vim /etc/minidlna.conf

    // 在配置文件内添加以下内容
    media_dir=P,/home/zqq/data/FilesSyncFromPhone/Redmi_K20_Pro_Premium_Edition-6fc19c2c/DCIM/Camera
    media_dir=A,/home/zqq/data/music
    media_dir=V,/home/zqq/data/视频

#### install kodi

    sudo apt install software-properties-common
    sudo add-apt-repository -y ppa:team-xbmc/ppa
    sudo apt install kodi

##### 解决访问文件夹的权限问题

*   表现：日志中打印Permission denied

1.  修改/etc/minidlna.conf中 user=root

2.  修改 /etc/default/minidlna 中USER="root"  GROUP="root"

3.  修改 /usr/lib/systemd/system/minidlna.service 中

    \[Service]
    User=root
    Group=roo

4.  重启服务

    sudo systemctl daemon-reload
    sudo service minidlna restart
    sudo service minidlna status

#### 安装音乐播放器clementine + gstreamer

    sudo apt install clementine

    sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5 gstreamer1.0-pulseaudio

#### 安装设置命令的默认程序update-alternatives 以python为例子

```shell
# 查看与python命令相关的执行程序，如果未配置则打印 错误: 无 python 的候选项
sudo update-alternatives --list python
# 安装默认程序 
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
# 设置默认程序
sudo update-alternatives --config python
```

#### 使用ffplay查看yuv文件

    //  -x 1280 -y 360 可指定显示的窗口大小
    ffplay -f rawvideo -pixel_format nv21 -video_size 1280x960 testCame_1280x960_2.yuv

#### 是用ffplay播放PCM文件

    ffplay -ar 44100 -channels 2 -f s16le -i ~/Downloads/capture_audio.pcm

#### 安装audacity用于播放编辑PCM（替代cooledit）

    sudo apt install audacity

#### 安装ctags

现在推荐安装universal-ctags。

    git clone https://github.com/universal-ctags/ctags.git
    cd ctags
    // checkout tag 
    git checkout p5.9.20220522.0
    ./autogen.sh
     ./configure 
     make
     sudo make install

或者直接安装已发布的版本

    sudo apt install universal-ctags

#### 安装特定版本内核

查看内核的更新记录，有助于判断是否是内核更新引入的问题
```
cat /var/log/dpkg.log | grep "linux-image"
```


[参考](https://zhuanlan.zhihu.com/p/133323571)
[命令行安装](https://www.cnblogs.com/carle-09/p/12377128.html#:\~:text=ubuntu---%E6%9F%A5%E7%9C%8B%E3%80%81%E5%AE%89%E8%A3%85%E3%80%81%E5%88%87%E6%8D%A2%E5%86%85%E6%A0%B8%20%E9%A6%96%E5%85%88%E5%8F%AF%E4%BB%A5%E6%9F%A5%E7%9C%8B%E4%B8%80%E4%B8%8B%E5%86%85%E6%A0%B8%E5%88%97%E8%A1%A8%EF%BC%9Asudo%20dpkg%20--get-selections%20%7C%20grep%20linux-image%20%E6%9F%A5%E7%9C%8BLinux%E4%B8%AD%E5%AE%89%E8%A3%85%E4%BA%86%E5%93%AA%E4%BA%9B%E5%86%85%E6%A0%B8%EF%BC%9A,%7C%20grep%20linux%20%E6%88%96%E8%80%85%20dpkg%20--list%20%7Cgrep%20linux)

1.  从[kernel ubuntu](https://kernel.ubuntu.com/\~kernel-ppa/mainline/)中下载到特定版本的内核文件。分别为linux-modules-xxx.deb linux-image-xxx.deb linux-headers-xxx.deb
2.  使用dpkg运行安装下载好的deb包

```shell
sudo dpkg -i linux-image-unsigned-5.15.0-051500rc7-generic_5.15.0-051500rc7.202110251930_amd64.deb linux-modules-5.15.0-051500rc7-generic_5.15.0-051500rc7.202110251930_amd64.deb
```
3.  安装完成后查看是否安装成功 ` sudo dpkg --get-selections | grep linux-image `

#### 安装当前的kernel header

<https://www.tecmint.com/install-kernel-headers-in-ubuntu-and-debian/#:~:text=Then%20run%20the%20following%20command%20that%20follows%20to,following%20command%20%24%20ls%20-l%20%2Fusr%2Fsrc%2Flinux-headers-%24%20%28uname%20-r%29>


#### 卸载内核版本
```
# 查看已安装的linux-image linux-module的具体版本号
sudo dpkg --list | grep linux-module
sudo dpkg --list | grep linux-image
# 根据上面的信息，删除具体的内核版本
sudo apt purge linux-image-xxx
sudo apt purge linux-module-xxx
```

#### 关闭、打开内核版本自动更新
```
sudo apt-mark hold linux-image-generic linux-headers-generic
sudo apt-mark unhold linux-image-generic linux-headers-generic
```

#### 切换TTY与打开关闭x server图像界面

    // 使用systemctl start、stop启动关闭
    sudo systemctl start gdm3
    sudo systemctl stop gdm3

切换TTY的快捷键：ctrl + alt + \[F1\~F7]

> F1~ F7 分别对应tty1，tty2，...， tty7。Ubuntu中tty1是用于显示桌面的。tty2~6是命令行。

如果使用sudo systemctl start gdm3关闭了桌面x server，此时可以先切换到tty2，再启动x server。从而恢复桌面显示。

#### 显示二进制的命令行

    hexdump -C <file>
    xxd  // 与hexdump -C效果类似

#### 观察CPU频率 功耗 温度的工具 s-tui

<https://zhuanlan.zhihu.com/p/385904157>

#### ubuntu更换清华源

[在线教程](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

    # 预发布软件源，不建议启用
    # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse

然后运行命令更新

    sudo apt-get update
    sudo apt-get upgrade

#### 查看文件、文件夾大小

```shell
# 查看文件的大小，以MB、KB、GB打印
ls -lh
# 查看文件夹的大小
du -h -x --max-depth=1 <dir>
# 查看大于1G的文件夹
du -lh | awk '{if(match($1, "G")>0) print $0}'
```

#### xargs的用法

##### 管道与xargs的区别

管道是实现“将前面的标准输出作为后面的标准输入”
xargs是实现“将标准输入作为命令的参数”

```shell
# 找到大于1G的文件夹，并ls -l
du -lh | awk '{if(match($1, "G")>0) print $2}' | xargs ls -l
```

## 网络相关
#### FTP服务

[安装教程](https://blog.csdn.net/weixin_45309916/article/details/107855067)

```
sudo apt install vsftpd
sudo vim /etc/vsftpd.conf
//修改配置项
local_enable=YES
write_enable=YES
// 重启服务
sudo /etc/init.d/vsftpd restart

```

Service的启动描述文件：/usr/lib/systemd/system/vsftpd.service

#### sftp客户端命令
```shell
# 使用用户名+主机ip或者主机名进行连接登录
sftp user_name@host
# 登录后直接通过ls、cd、pwd、mkdir命令操作目录
操作远程主机目录：ls、cd、pwd、mkdir、rm、chmod
操作本地主机目录：lls、lcd、lpwd、lmkdir、lrm、lchmod
# 上传文件到远程服务器
put 本地文件 [远程路径]	
# 从远程服务器下载文件
get 远程文件 [本地路径]
# 使用通配符传输多个文件
mput *.jpg      # 上传所有jpg文件
# 下载整个目录（递归）
sftp -r user@example.com:/backups /local/backups
```

#### 网络命令汇总

    //显示路由表
    route -n
    //查看网络流量
    vnstat -i eth0 -l
    // 查看进程使用的流量信息
    iftop -i eth0
    // 查看端口占用
    netstat -anp | grep 6039

### 使用nmcli进行网络配置
[简单教程](https://www.runoob.com/linux/linux-comm-nmcli.html)
常用命令
```sh
nmcli device status
# 显示所有连接状态
nmcli connection show
# 显示单个连接详细信息
nmcli connection show <id>
# 启动与停止连接
nmcli connection up <id>
nmcli connection down <id>
# 修改connection的单个属性值
nmcli connection modify <id> ipv4.method <shared | auto>
systemctl restart NetworkManager
```

#### WiFi无法上网
sudo systemctl status wpa_supplicant.service
sudo systemctl restart wpa_supplicant.service

This solved the error with my RTL8188 USB WiFi adapter.
And changed the /usr/lib/systemd/system/wpa_supplicant@.service accordingly.

ExecStart=/usr/bin/wpa_supplicant -Dwext -c/etc/wpa_supplicant/wpa_supplicant-%I.conf -i%I

+ 使用`sudo dmesg |grep -i iwlwifi`看到以下错误信息：
FW error in SYNC CMD REDUCE_TX_POWER_CMD
***

#### 虚拟机性能

1.  将系统的swap空间扩大到8G以上，给显示用。
2.  关闭windows的虚拟分页

#### 启用中文输入法或者搜狗输入法

[参考](https://cloud.tencent.com/developer/article/1623270)

[搜狗输入法安装](https://shurufa.sogou.com/linux/guide)
[下载搜狗输入法deb](https://shurufa.sogou.com/)

#### 安装VPN支持

[教程](https://blog.csdn.net/huaxiao137/article/details/88715427)
[l2tp.sh下载](https://github.com/teddysun/across)

#### Ubuntu更该启动顺序

启动相关grub2主要包含下面三个文件：1.   /boot/grub/grub.cfg 文件    2.   /etc/grub.d/ 文件夹   3.   /etc/default/grub 文件，可以通过修改这三个文件来修改启动项

    sudo gedit /etc/default/grub
    //GRUB_DEFAULT=0     #更改数字设置默认启动项
    sudo update-grub
    sudo reboot 

#### TP-Link的无线网卡驱动

[驱动下载](https://github.com/aircrack-ng/rtl8814au)

***

## 显卡

查看所有显卡设备

    lspci -nn |grep  -Ei 'VGA|DISPLAY'

### AMD显卡驱动安装

[参考](https://help.ubuntu.com/community/BinaryDriverHowto)\
[下载地址1](https://www.amd.com/en/support/kb/release-notes/rn-rad-lin-13-12)\
[下载地址2](http://repo.radeon.com/amdgpu-install/latest/ubuntu/focal/)

> 安装了驱动之后，虚拟机可能无法启动。OpenGL无法创建Render
> 解决方式：在.vmware/preferences中添加:\
> \> mks.gl.allowUnsupportedDrivers="TRUE"

### intel显卡驱动安装

[intel hardware-table](https://dgpu-docs.intel.com/devices/hardware-table.html)
[官方安装指引，但没有早期ubuntu系统的](https://dgpu-docs.intel.com/driver/client/overview.html#installing-client-gpus-on-ubuntu-desktop-22-04-lts)
[How to Install the Intel Graphic Drivers on Ubuntu ](https://www.linuxfordevices.com/tutorials/ubuntu/install-intel-graphic-drivers#:\~:text=You%20can%20download%20the%20Intel%20graphic%20drivers%20by,to%20update%20your%20package%20list%3A%20sudo%20apt-get%20update.)

#### 安装intel驱动

1.  先试用lspci -nn | grep -Ei 'VGA|DISPLAY' 查看intel显卡硬件是否检测得到。检测不到去BIOS的Graphics Configuration里面开启IGFX显卡

#### 查看intel GPU使用率

    sudo apt-get install intel-gpu-tools
    sudo intel_gpu_top

1.  安装

##### ubuntu 22.04下安装

    # Install the Intel graphics GPG public key
    wget -qO - https://repositories.intel.com/gpu/intel-graphics.key | \
      sudo gpg --yes --dearmor --output /usr/share/keyrings/intel-graphics.gpg

    # Configure the repositories.intel.com package repository
    echo "deb [arch=amd64,i386 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu jammy client" | \
      sudo tee /etc/apt/sources.list.d/intel-gpu-jammy.list

    # Update the package repository meta-data
    sudo apt update

    # Install the compute-related packages
    apt-get install -y libze1 intel-level-zero-gpu intel-opencl-icd clinfo libze-dev intel-ocloc

##### ubuntu 20.04下安装

    wget -qO - https://repositories.intel.com/graphics/intel-graphics.key | \
      sudo gpg --yes --dearmor --output /usr/share/keyrings/intel-graphics.gpg

    echo "deb [arch=amd64,i386 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu focal client" | \
      sudo tee /etc/apt/sources.list.d/intel-gpu-focal.list
      
    sudo apt update

    sudo apt install \
      intel-opencl-icd \
      intel-level-zero-gpu level-zero \
      intel-media-va-driver-non-free libmfx1
      
    sudo apt install \
      libigc-dev \
      intel-igc-cm \
      libigdfcl-dev \
      libigfxcmrt-dev \
      level-zero-dev

1.  检查

<!---->

    clinfo | grep "Device Name"

You should see the Intel graphics product device names listed. If they do not appear, ensure you have permissions to access /dev/dri/renderD\*. This typically requires your user to be in the render group:

    sudo gpasswd -a ${USER} render
    newgrp render

### 在intel集成显卡与nvidia显卡之间切换

        sudo prime-select nvidia
        sudo prime-select intel

### NVIDIA显卡

#### 查看信息

[ubuntu查看显卡信息、卸载驱动、CUDA](https://zhuanlan.zhihu.com/p/243256494)

    // 查看显卡型号
    $ lspci | grep -i nvidia（-i表示不区分大小写）
    // 查看显卡详细信息
    nvidia-smi -L （必须安装好nvidia驱动）
    // 查看系统内核兼容的显卡驱动版本
    cat /proc/driver/nvidia/version
    // 查看系统安装的显卡驱动版本，对应显示结果中libnvidia-common的版本
    dpkg -l | grep nvidia

#### 安装nvidia驱动

[参考资料](https://help.ubuntu.com/community/BinaryDriverHowto/Nvidia#Troubleshooting)
[nvidia的教程--有问题时很有帮助](http://us.download.nvidia.cn/XFree86/Linux-x86_64/535.54.03/README/installdriver.html)

前提：

1.  安装linux-kernel-header
    [安装方法参考](https://www.tecmint.com/install-kernel-headers-in-ubuntu-and-debian/#:\~:text=Then%20run%20the%20following%20command%20that%20follows%20to,following%20command%20%24%20ls%20-l%20%2Fusr%2Fsrc%2Flinux-headers-%24%20%28uname%20-r%29)

<!---->

        uname -r
        apt search linux-headers-`$(uname -r)
        sudo apt install linux-headers-$`(uname -r)

1.  确保之前可能安装失败的NVIDIA驱动已经卸载了。
    > 再卸载一次

方式1：可以直接通过命令行安装（但试过几次都失败了）

    sudo apt install nvidia-driver-460
    # 或者安装nvidia-driver-530-open
    sudo apt install nvidia-driver-530-open
    sudo apt install nvidia-kernel-open-530

方式2：NVIDIA官网下载安装包.run文件
==在安装run文件之前需要先判断内核使用的gcc版本，当前的编译环境最好切换到与内存gcc版本一致。因为run文件安装过程需要使用gcc进行代码编译，此时如果当前gcc版本与内核的gcc版本不一致会报错。切换gcc版本的方式可以查看上面。==
[参考](https://zhuanlan.zhihu.com/p/620214740)

    # 先切换到ubuntu官方的源
    #更新软件列表
    sudo apt-get update
    sudo apt-get install g++ gcc make
    # 禁用 nouveau
    sudo gedit /etc/modprobe.d/blacklist.conf
    # 在最后两行添加
    blacklist nouveau
    options nouveau modeset=0
    # 重启系统
    # 进入命令行界面
    sudo telinit 3
    # 关闭显示服务
    sudo service gdm3 stop
    # 赋予可执行权限，开始安装
    sudo chmod 777 NVIDIA-Linux-x86_64-525.60.11.run 
    sudo ./NVIDIA-Linux-x86_64-525.60.11.run
    # 如果安装失败可以在/var/log/nvidia-installer.log
    # 安装完成后重新开启显示服务
    sudo service gdm3 start
    # 确认安装是否成功
    nvidia-smi

安装后可以通过xorg.conf配置针对nvidia的特殊使用。
==在内核6.8.0安装驱动失败了==
错误信息：/tmp/selfgz5163/NVIDIA-Linux-x86\_64-535.104.05/kernel/nvidia-drm/nvidia-drm-drv.c:1290:40: error: 'DRM\_UNLOCKED' undeclared here
对比6.8.0内核DRM的头文件会发现 DRM\_UNLOCKED已经被删除了。比如对比6.5.0与6.8.0
/usr/src/linux-hwe-6.8-headers-6.8.0-40/include/drm/drm\_ioctl.h
/usr/src/linux-hwe-6.5-headers-6.5.0-44/include/drm/drm\_ioctl.h

如果想要重新安装则先进行驱动卸载

    sudo apt remove nvidia-*
    sudo apt purge nvidia-*
    sudo /usr/bin/nvidia-uninstall（亲测可以）

#### 安装AMD显卡驱动

1.  先从官网下载对应的驱动安装包。压缩包
2.  解压后有安装脚本

\==**新的内核中，如果没有安装显卡驱动可能无法启动，此时先用旧的内核版本，启动后安装驱动即可使用**==

#### 使用diff

```bash
#只显示差异的地方
diff log2014.log log2013.log

#并排格式输出，相同的行也显示 -W可以不用
diff log2014.log log2013.log  -y -W 50

# --strip-trailing-cr为忽略换行符差异 -B为忽略空白行
diff --strip-trailing-cr mediasessionjob.cpp mediajobbase-unify-nimo.cpp -y -W 180 -B
```

*   "|"表示前后2个文件内容有不同
*   "<"表示后面文件比前面文件少了1行内容
*   ">"表示后面文件比前面文件多了1行内容

#### 将文件内容转化成16进制显示

使用xxd 或者 od 命令

```bash
xxd file

#在vi命令状态下：
:%!xxd :%!od 将当前文本转化为16进制格式
:%!xxd -c 12 每行显示12个字节
:%!xxd -r    将当前文本转化回文本格式
```

#### 屏幕截图

PrtSc为截屏快捷键，加Shift截取区域，加Alt则是截取窗口。默认保存在Picture目录。
如果需要保存到剪贴板则加CTRL。

*   PrtSc – 获取整个屏幕的截图并保存到 Pictures 目录。
*   Shift + PrtSc – 获取屏幕的某个区域截图并保存到 Pictures 目录。
*   Alt + PrtSc –获取当前窗口的截图并保存到 Pictures 目录。
*   Ctrl + PrtSc – 获取整个屏幕的截图并存放到剪贴板。
*   **Ctrl + Shift + PrtSc – 获取屏幕的某个区域截图并存放到剪贴板。**
*   *Ctrl + Alt + PrtSc – 获取当前窗口的 截图并存放到剪贴板。*\*

#### linux虚拟机提示无法加载显卡驱动

当把nvida卡拔掉，改用集成显卡时，VM提示显卡性能不足
修改方式：

    vim .vmware/preferences   //修改VM配置文件
    mks.gl.allowBlacklistedDrivers = "TRUE"

#### 添加国内的镜像软件源

[清华](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

#### ubuntu安装deb包

    sudo dpkg -i '/tmp/mozilla_zqq0/google-chrome-stable_current_amd64.deb'
    // 如果安装过程中发现依赖的库或者软件未安装，则可以
    sudo apt -f install

#### 修改PATH环境变量

    // 修改 ~/.bashrc 添加
    export PATH=/home/zqq/snap/android-studio-ide-191.6010548-linux/android-studio/bin/:$PATH
    // 命令行执行
    source ~/.bashrc

#### 查看磁盘信息

    sudo fdisk -l
    // 查看分区UUID
    sudo blkid

#### 查看所有磁盘挂载点

    df -kh
    df -lh  //显示更详细，包括磁盘占用率

#### 创建新的硬盘分区或者格式化硬盘[参考](https://zhuanlan.zhihu.com/p/397440213)

    sudo fdisk /dev/sdb    #给硬盘sdb创建分区
    # 上面的命令进入一个命令界面，在命令界面内
    p -- 查看分区信息  m--查看帮助  d--删除分区 n--创建新分区
    sudo mkfs.ext4 /dev/sdb1    #格式化硬盘sdb，并写入文件系统

#### 临时挂载磁盘到某个目

    sudo mount /dev/sda2 ~/D
    //卸载
    sudo umount /dev/sda2

#### 永久性挂载分区

    // 编辑fstab文件
    sudo gedit /etc/fstab
    //先打开另一个终端，找到/dev/sda2分区对应的UUID
    sudo blkid /dev/sda2

我们按照/etc/fstab文件中的格式添加一行如下内容:\
其中第一列为UUID, 第二列为挂载目录（该目录必须为空目录），第三列为文件系统类型，第四列为参数，第五列0表示不备份，最后一列必须为２或0(除非引导分区为1

    UUID=0001D3CE0001E53B /home/ubuntu/NewDisk ntfs defaults 0 2 

运行命令或者重启电脑

    sudo mount -a

#### 查找

##### 查找文件并使用ls查看

```shell
    find ./ -name "*.so" -exec ls -l {} \;
    # 在当前目录下排除abc目录，查找所有以.txt结尾的文件【方式一】
    find . -path "./abc" -prune -o -name "*.txt" -print
    # 查找特定的文件，并grep文件内带有特定字符串的语句
    find . -name CMakeLists.txt -exec ls -l {} | grep ffmpeg {} \;
```

#### 关于安装l2tp、ipsec支持VPN

*   [网络教程](https://blog.csdn.net/huaxiao137/article/details/88715427)

##### 启动服务

    ># systemctl   start  ipsec   
    ># systemctl  start l2tpd

#### 查看文件内容

<https://blog.csdn.net/jeikerxiao/article/details/77503479>

### apt

#### 查找头文件关联的库并安装

    apt search Xlib.h
    sudo apt install libx11-dev

### 设置修改swap分区（文件）

[来源](https://help.ubuntu.com/community/SwapFaq)

Create the Swap File:
We will create a 1 GiB file (/mnt/1GiB.swap) to use as swap:

    sudo fallocate -l 1g /mnt/1GiB.swap

fallocate size suffixes: g = Giga, m = Mega, etc. (See man fallocate).

If fallocate fails or it not available, you can use dd:

    sudo dd if=/dev/zero of=/mnt/1GiB.swap bs=1024 count=1048576

We need to set the swap file permissions to 600 to prevent other users from being able to read potentially sensitive information from the swap file.

    sudo chmod 600 /mnt/1GiB.swap

Format the file as swap:

    sudo mkswap /mnt/1GiB.swap

Enable use of Swap File

    sudo swapon /mnt/1GiB.swap

The additional swap is now available and verified with:

    cat /proc/swaps

Enable Swap File at Bootup
Add the swap file details to /etc/fstab so it will be available at bootup:

    echo '/mnt/1GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab

### 查看各种信息
```shell
    // 查看内核版本信息
    uname -r
    uname -a
    // 查看linux操作系统版本信息
    cat /etc/*release
    lsb_release -a
    cat /etc/issue
    // 查看CPU信息
    lscpu
    // 查看有线网卡型号
    lspci | grep -i ethernet
    // 查看内存
    free -h
```
#### arm或者嵌入式linux系统查看各种信息
[Android使用adb命令查看APP数据流量使用情况](https://www.cnblogs.com/liyuanhong/articles/11376302.html#:~:text=%E5%9C%A8Android%E7%B3%BB%E7%BB%9F%E4%B8%AD%EF%BC%8C%22%2Fproc%2Fnet%2Fxt_qtaguid%2Fstats%22%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E9%87%8C%E5%82%A8%E5%AD%98%E7%9D%80%E5%90%84%E4%B8%AA%E5%BA%94%E7%94%A8%E7%9A%84%E6%B5%81%E9%87%8F%E4%BF%A1%E6%81%AF%EF%BC%8C%E7%BB%9F%E8%AE%A1%E6%B5%81%E9%87%8F%E7%9A%84%E6%97%B6%E5%80%99%E5%8F%AF%E4%BB%A5%E7%94%A8%E5%88%B0%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E3%80%82,android4.0%E4%BB%A5%E4%B8%8A%E7%89%88%E6%9C%AC%E5%8F%AF%E4%BB%A5%E7%94%A8%2Fproc%2Fuid_stat%2F%24uid%2Ftcp_rcv%E5%92%8C%2Fproc%2Fuid_stat%2F%24uid%2Ftcp_snd%E6%9D%A5%E8%8E%B7%E5%8F%96%E6%9F%90%E4%B8%AA%E7%A8%8B%E5%BA%8F%E7%9A%84%E4%B8%8A%E4%B8%8B%E8%A1%8C%E6%B5%81%E9%87%8F%EF%BC%9B%E8%80%8C4.0%E4%BB%A5%E4%B8%8B%E7%89%88%E6%9C%AC%E8%A6%81%E7%94%A8cat%2Fproc%2F%24pid%2Fnet%2Fdev%E6%9D%A5%E6%9F%A5%E7%9C%8B%E4%B8%8A%E4%B8%8B%E8%A1%8C%E6%B5%81%E9%87%8F%E3%80%82)
```shell
# 查看CPU使用率
top
# 查看GPU使用率 比如rknn、gpu
cat /sys/class/devfreq/fb000000.gpu/load
# 查看NPU使用率 比如rknn、gpu
cat /sys/class/devfreq/fdab0000.npu/load
# 查看网络进出流量
 cat /proc/pid/net/dev
 cat /sys/class/net/wlan0/statistics/rx_bytes
```

[关于查看内存硬件信息 参考](https://blog.csdn.net/kaikai_sk/article/details/84752550)

### linux支持调试Android设备、APP

Ubuntu Linux：需要正确进行两项设置：希望使用 adb 的每个用户都需要位于 plugdev 组中，并且需要为系统添加涵盖设备的 udev 规则。

plugdev 组：如果您看到一条错误消息，指出您不在 plugdev 组中，您需要将自己添加到 plugdev 组中：

```bash
sudo usermod -aG plugdev $LOGNAME
```

请注意，组只会在您登录时更新，因此您需要退出后重新登录，此更改才能生效。当您重新登录后，可以使用 id 检查自己现在是否已在 plugdev 组中。----实测，ubuntu.20可以不用重登陆

udev 规则：android-sdk-platform-tools-common 软件包中包含一组适用于 Android 设备并由社区维护的默认 udev 规则。请使用以下命令添加这些规则：

```bash
apt-get install android-sdk-platform-tools-common
```

### 关于安装intel显卡驱动

[参考](https://dgpu-docs.intel.com/installation-guides/ubuntu/ubuntu-focal.html)

### 查看so共享库 .a静态库的导出符号

    nm -D xxx.so
    # 得到的结果中以T开头的就是导出函数

    # 列出所有定义的符号
    nm --defined-only liba.so

    objdump -tT xxx.so

### 查看设备信息

    # 查看usb端口与设备列表。“3.0 root hub”表明USB是3.0的
    lsusb

#### 安装ubuntu 22.04后安装的其他deb软件无法打开，比如code或者edge

解决方法：删掉snap

    for p in $( snap list | awk '{print $1}' ); do
        sudo snap remove --purge $p
    done

    sudo apt remove snapd
    # 创建文件 /etc/apt/preferences.d/nosnap.pref
    sudo vim /etc/apt/preferences.d/nosnap.pref
    # 文件内容如下
    Package: snapd
    Pin: release a=*
    Pin-Priority: -10

    # 更新
    sudo apt update
    # 重其看看

    sudo rm -rvf /var/lib/apt/lists/*
    sudo apt update
    reboot

### 安装xvfb

xvfb是linux下的虚拟屏幕。当程序运行在linux server，且需要使用OpenGL渲染功能时，可以安装此虚拟屏幕软件。不然的话eglInitialize就可能初始化失败。

    Xvfb :98 -screen 0 1336x768x24 2>/dev/null &
    export DISPLAY=:98

### 安装x-server环境

[ubuntu serverGUI](https://help.ubuntu.com/community/ServerGUI)

    sudo apt-get install fglrx
    sudo apt-get install xserver-xorg-core xserver-xorg xorg xauth openbox

### 安装x-client
```
    suo apt install ubuntu-desktop
```

### 解决机器启动后无法进入UI界面
表现：开机启动后，无法进入UI登录界面，卡在输出内核加载信息的过程打印中，一直无法进入。
原因：显卡驱动与nouveau驱动不兼容导致无法加载显卡驱动。显卡是NVIDIA的。
处理方式：
1. 重启电脑，在 GRUB 启动菜单选择 Advanced options for Ubuntu
2. 选择 Recovery mode
3. 选择 root (Drop to root shell prompt)。这样可以进入命令行模式。
4. ==完全移除 Nouveau 驱动。==(这步很重要)  更新 initramfs
```shell
# 删除所有 Nouveau 相关包
sudo apt purge xserver-xorg-video-nouveau libdrm-nouveau2

# 确保 Nouveau 被列入黑名单
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
# 更新初始化内存盘
sudo update-initramfs -u
# 设置grub完全禁用nouveau
# 编辑 GRUB 配置
sudo nano /etc/default/grub

# 找到 GRUB_CMDLINE_LINUX_DEFAULT 行，添加：
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nouveau.modeset=0"

# 更新 GRUB
sudo update-grub
```
5. 重新安装NVIDIA显卡驱动，可以不用卸载再装，直接覆盖。
6. 重启系统。

***

## 编程工具

#### 查看so或者程序依赖库

```bash
readelf -d xxx.so | grep NEEDED
```

[参考资料](https://unix.stackexchange.com/questions/120015/how-to-find-out-the-dynamic-libraries-executables-loads-when-run)
[参考资料2](https://blog.csdn.net/mayue_web/article/details/104019036)

#### 查看so或者可执行程序是否可调试，是否可debug

    readelf -S libYCloudLive.so | grep debug 

#### 查看so或者可执行程序的依赖

    # 方法1，推荐
    ldd xxx.so
    # 方法2
    readelf -d libbuglyversion.so | grep 'NEEDED'

#### 设置程序运行时先从特定目录链接so

    # 比如设置从shell当前目录查找、链接so
    export LD_LIBRARY_PATH=./
    # 使用相对路径可能失效，推荐使用绝对路径

#### valgrind

[参考1](https://zhuanlan.zhihu.com/p/107120029)

    valgrind --log-file=memcheck-9.log --tool=memcheck --leak-check=full --show-leak-kinds=all ./magic_server magic_server.cfg

*   结束执行后，查看log文件。
*   搜索关键字：'definitely lost'。这个表示确定的内存泄漏

#### 调试、崩溃

##### 使用addr2line打印崩溃堆栈的代码路径

    addr2line -Cfpie hymediatrans-debug-1.25.33-rel-cloudgame-server-arm64-v8a.so 7095c 

此命令需要先按照binutils

    sudo apt install binutils

#### 崩溃时coredump的设置与获取

通过ulimit设置启用在程序崩溃时自动生成dump，大小无限制。通过sysctl设置保存dump文件的路径。

    ulimit -c unlimited
    sudo sysctl -w kernel.core_pattern=/data/coredump/core.%u.%e.%p

手动生成进程的coredump文件

    gcore [-o filename] pid

***

## SSH

1.  服务端需要安装 openssh-server，客户端则安装 openssh-client (ubuntu默认安装了)

2.  服务端启动与关闭服务

    sudo /etc/init.d/ssh start  // 启动
    sudo /etc/init.d/ssh stop   // 停止
    sudo /etc/init.d/ssh restart   // 重启

3.  客户端登陆操作
    *   linux使用 ssh name\@ip 进行登陆ssh server。需要输入用户名与密码。

4.  客户端退出ssh登陆时服务器主机不中断运行程序。 \$ nohup \[程序]

    nohup python -u test.py > ./log.txt 2>&1 &

5.  使用scp进行文件传输

    scp -P 56000 -r audio-driving-linux-server-dev.zip \_\_su\@10.112.205.66:/data/audio\_driving\_dev

1\.

***

## nextcloud

#### ubuntu使用docker安装。[参考](https://github.com/nextcloud/all-in-one#nextcloud-all-in-one)

    #安装docker
    curl -fsSL https://get.docker.com | sudo sh

    #启动nextcloud
    # For Linux and without a web server or reverse proxy (like Apache, Nginx, Cloudflare Tunnel and else) already in place:
    sudo docker run \
    --sig-proxy=false \
    --name nextcloud-aio-mastercontainer \
    --restart always \
    --publish 80:80 \
    --publish 8080:8080 \
    --publish 8443:8443 \
    --volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    nextcloud/all-in-one:latest

*   docker内跑起nextcloud，可以直接用浏览器输入https\://\[ip]:8080，进行访问，然后登陆，AIO Password
    employer chrome say recall stream maimed residency wildfire

***

### 进程相关

    # 按名称找到进程
    ps -ef | grep 'nginx'
    # 打印进程详细信息，并找到进程启动的路径
    # exe -> /usr/local/nginx/sbin/nginx exe指示的就是执行的程序的路径
    ls -l /proc/[pid]
    # 查看所有进程信息top。
    # 进入top后，按1可以查看CPU每个核的使用率，按e可以切换内存的单位
    top
    # 查看进程的所有线程使用的资源（包括cpu使用率）
    top -H -p <pid>

##### SHELL 多个命令一起执行的几种方法

    # 每个命令之间用;隔开.各命令的执行结果，不会影响其它命令的执行
    cd /home/PyTest/src; python suning.py
    # .每个命令之间用&&隔开. 若前面的命令执行成功，才会去执行后面的命令
    cd /home/PyTest/src&&python suning.py
    # 每个命令之间用||或者|隔开. ||是或的意思，只有前面的命令执行失败后才去执行下一条命令，直到执行成功
    一条命令为止。
    cd /home/PyTest/123 || echo "error234"

##### linux下的守护进程daemon

    // 启动进程并守护 start-stop-daemon --start --quiet --make-pidfile --pidfile <pid_file> --exec <command>
    start-stop-daemon --start --quiet --make-pidfile --pidfile ~/xvfb.pid --exec /usr/bin/Xvfb -- :99 -screen 0 1920x1080x24 -ac &
    // 关闭进行并停止守护
    start-stop-daemon --stop --name Xvfb

##### 清理
```shell
    #清理.cache文件夹下超过90天的文件
    find ~/.cache/ -type f -atime +90 -delete
    # 清理/var/log/syslog
    sudo rm -f /var/log/syslog
    sudo systemctl restart rsyslog.service
    # 如果要限制syslog大小，则可以修改文件/etc/logrotate.d/rsyslog
```

##### 解决sudo部分命令找不到的问题

比如sudo ncu将找不到ncu命令，执行以下命令

    sudo visudo
    # 将ncu命令所在的目录添加到Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:"内

#### ubuntu版本升级

<https://zhuanlan.zhihu.com/p/694832548>

*   [linux-firmware硬件固件](https://gitlab.com/kernel-firmware/linux-firmware)
```
# 手动更新firmware
sudo cp -r ~/linux-firmware/* /lib/firmware/
```

#### ubuntu删除安装失败了的软件包

    sudo dpkg --remove --force-remove-reinstreq package_name 
    sudo apt-get update

#### thinkbook 16+ 2024驱动

*   [触摸板驱动](https://github.com/ty2/goodix-gt7868q-linux-driver)
*   如果是ubuntu则改为将quirks复制到 /usr/share/libinput/&lt;filename&gt;.quirks [参考](https://wayland.freedesktop.org/libinput/doc/latest/device-quirks.html)

#### 系统备份与恢复

1.  使用remastersys工具进行备份，系统将备份成iso，然后再制作成安装启动U盘

<!---->

    sudo apt install dialog casper libdebian-installer4
    sudo dpkg -i remastersys-xxx

1.  remastersys创建iso文件总是失败，即使选择只生成cdfs的文件也会失败。暂时放弃了
2.  [参考](https://wiki.ubuntu.com.cn/Remastersys)

改为使用systemback进行备份

1.  安装与备份
2.  创建iso
3.  恢复安装系统

### 内核模块管理

#### 开机时自动加载特定模块，比如v4l2loopback

在/etc/modules文件末尾追加模块名称

    v4l2loopback

然后执行：

    sudo update-initramfs -c -k $(uname -r)

#### 查看所有加载的模块

    lsmod  

### 安装字体

直接把ttf之类的字体文件拷贝到/usr/share/fonts下

    sudo cp ~/Downloads/*.ttf /usr/share/fonts
    sudo fc-cache -f -v

#### 字体文件的配置文件
`/etc/fonts/conf.d/64-language-selector-prefer.conf`
这个文件中配置了默认的字体，比如中文、日文、韩文等。
在使用libass库时，如果渲染是指定了font family，但是某个字符在指定的font family的字体文件中找不到graphy，则libass一般会自动回退到使用此配置文件配置的字体进行渲染。如果此配置文件中配置的字体不存在或者也不包含那个graphy，则可能会渲染乱码。

#### 开源字体
[noto-cjk](https://github.com/notofonts/noto-cjk)
[noto](https://github.com/notofonts/notofonts.github.io/)

### 屏幕显示相关

#### 屏幕旋转后配置触控屏

编辑修改 /usr/share/X11/xorg.conf.d/40-libinput.conf,找到文件里面的touchscreen这个section。添加一行：

    # 90度
    Option "CalibrationMatrix" "0 1 0 -1 0 1 0 0 1"
    # 180度
    Option "CalibrationMatrix" "-1 0 1 0 -1 1 0 0 1"
    # 270度
    Option "CalibrationMatrix" "0 -1 1 1 0 0 0 0 1"
    # 同时翻转左右与上下
    Option "CalibrationMatrix" "-1 0 1 0 -1 1 0 0 1"

触摸屏位置不准确的调节

    sudo apt-get install xinput-calibrator

#### 工作区配置
1. 安装gnome-tweaks通过GUI进行控制
```shell
# 安装
sudo apt install gnome-tweaks
# 直接命令行输入启动
gnome-tweaks
```

2. 通过命令gsettings设置
```shell
# 获取gnome是否支持动态工作区个数
gsettings get org.gnome.mutter dynamic-workspaces
# 配置是否开启动态工作区 true--开启 false--关闭
gsettings set org.gnome.mutter dynamic-workspaces true
# 设置与获取工作区个数，可以设置为1，然后通过配置
gsettings get org.gnome.desktop.wm.preferences num-workspaces
gsettings set org.gnome.desktop.wm.preferences num-workspaces 1
```

#### 禁用触摸屏或者触摸板的手势动作
```shell
# 禁用触摸屏手势动作
gsettings set org.gnome.desktop.peripherals.touchscreen:/usr/share/glib-2.0/schemas/ send-events 'disabled'
# 上面命令可能因为/usr/share/glib-2.0/schemas/下没有org.gnome.desktop.peripherals.touchscreen相关的配置而失败
# 此时可以创建文件 sudo vim /usr/share/glib-2.0/schemas/99-custom-touchscreen.gschema.override
sudo vim /usr/share/glib-2.0/schemas/99-custom-touchscreen.gschema.override
# 文件内容如下：
[org.gnome.desktop.peripherals.touchscreen]
send-events='disabled'
# 重新编译gschemas
sudo glib-compile-schemas /usr/share/glib-2.0/schemas/
# 这样编译也可能出现如下错误：
# No such key “send-events” in schema “org.gnome.desktop.peripherals.touchscreen” as specified in override file “/usr/share/glib-2.0/schemas/99-custom-touchscreen.gschema.override”; ignoring override for this key.
# 这种情况下可以通过grep touchscreen /usr/share/glib-2.0/schemas/*找到touchscreen是在哪个文件配置，然后修改那个文件的配置。比如：
sudo vim /usr/share/glib-2.0/schemas/org.gnome.desktop.peripherals.gschema.xml
#在touchscreen相关的配置中添加以下内容：
    <key name="send-events" enum="org.gnome.desktop.GDesktopDeviceSendEvents">
      <default>'enabled'</default>
      <summary>Touchpad enabled</summary>
      <description>Defines the situations in which the touchpad is enabled.</description>
    </key>
# 再次complie schemas
sudo glib-compile-schemas /usr/share/glib-2.0/schemas/
# 通过list-keys、get确认修改生效
gsettings list-keys org.gnome.desktop.peripherals.touchscreen:/usr/share/glib-2.0/schemas/
gsettings get org.gnome.desktop.peripherals.touchscreen:/usr/share/glib-2.0/schemas/ send-events
```

#### gsettings的其他命令
```shell
# 列出所有的schema
gsettings list-schemas
# 获取schemas下面的所有的key
gsettings list-keys org.gnome.desktop.peripherals.touchpad
# 获取schemas下的keys的值
gsettings list-recursively org.gnome.desktop.peripherals.touchpad
```

### 禁止特定用户使用sudo -i或者sudo su
以root修改文件内容 /etc/sudoers
```shell
sudo visudo
# 修改以下内容
xmagic ALL=(ALL) NOPASSWD: !/bin/bash, !/bin/sh, !/bin/su, !/usr/bin/sudo -i, !/usr/bin/sudo su
```
### 硬盘深拷贝
```
    可先用`fdisk -lh`查看需要复制的磁盘和目标磁盘
    sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress  
```
### Ubuntu永久获得串口权限。串口usb读写权限

    # 创建rule文件
    sudo vim /etc/udev/rules.d/70-ttyUSB.rules
    # 添加以下内容
    KERNEL=="ttyUSB*", OWNER="root", GROUP="root", MODE="0666"
    # 保存然后修改rules文件权限
    sudo chmod 666 /etc/udev/rules.d/70-ttyUSB.rules
    # 重启系统

## 开机自动启动程序
### 在系统中安装与管理服务service
```

sudo vim /etc/systemd/system/service\_daemon.service

# 生效
sudo systemctl daemon-reload
# 设置开机启动
sudo systemctl enable service\_daemon.service
# 立即启动
sudo systemctl start service\_daemon.service
\#检测服务状态
sudo systemctl status service\_daemon.service
# 停止
sudo systemctl stop service\_daemon.service
# 禁用开机启动
sudo systemctl disable service\_daemon.service
```
### 在~/.config/autostart中添加可执行的程序或者脚本
```
# 拷贝已安装的程序的快捷方式
cp /usr/share/applications/tmeta.desktop ~/.config/autostart
sudo chmod a+x ~/.config/autostart/tmeta.desktop
```

### 设置开机自动登录
修改文件：`/etc/gdm3/custom.conf`
```
# 修改以下内容 true--开启自动登录 false--关闭自动登录
[daemon]
AutomaticLoginEnable=true
```

sudo 
***
### 设置生成coredump

    ulimit -c unlimited  # 不限制core文件大小，能大概率确保可以生成core文件。这个是临时设置，机器重启后将失效
    # 使用GDB调试coredump文件
    gdb app
    # 进入调试界面后
    core-file <core file path>
    # 修改/proc/sys/kernel/core_pattern文件内容设置core文件的保存路径和命名规则
    echo /usr/core_dump/core_%e_%t_%p > /proc/sys/kernel/core_pattern   # 这个设置是临时的

    # 永久配置core文件unlimited和保存路径、名称
    # 修改 /etc/security/limits.conf，添加下面2行
    @root soft core unlimited
    @root hard core unlimited
    # 修改/etc/sysctl.conf文件，添加下面2行：
    kernel.core_pattern = /usr/core_dump/core_%e_%t_%p
    kernel.core_uses_pid = 0

### 周期性清理日志或者过期文件
#### crontab的用法--[参考](https://www.runoob.com/linux/linux-comm-crontab.html)
```
# 查看当前用户的 crontab 文件
crontab -l
# 编辑当前用户的 crontab 文件
crontab -e
```
直接使用`crontab -e`编辑crontab文件，添加task
```
# 比如
# m h  dom mon dow   command
  * *  *   *   *     echo "start clear log" >> /home/xmagic/crontab.log
#  * *  *   *   *     find /home/xmagic/runtime/magic_immd_server/*/logs -type f -atime +15 -print0 | xargs -0 -n 100 echo >> /home/xmagic/crontab.log
  0 *  *   *   *     find /home/xmagic/runtime/magic_immd_server/*/logs -type f -atime +15 -print0 | xargs -0 -n 100 rm -f
```