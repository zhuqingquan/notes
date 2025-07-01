## 编码器的名称
可用于-vcodec参数的值
 V..... libx264              libx264 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (codec h264)
 V..... libx264rgb           libx264 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 RGB (codec h264)
 V....D h264_amf             AMD AMF H.264 Encoder (codec h264)
 V....D h264_nvenc           NVIDIA NVENC H.264 encoder (codec h264)
 V..... h264_qsv             H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (Intel Quick Sync Video acceleration) (codec h264)
 V..... nvenc                NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc_h264           NVIDIA NVENC H.264 encoder (codec h264)
 V..... libx265              libx265 H.265 / HEVC (codec hevc)
 V..... nvenc_hevc           NVIDIA NVENC hevc encoder (codec hevc)
 V....D hevc_amf             AMD AMF HEVC encoder (codec hevc)
 V....D hevc_nvenc           NVIDIA NVENC hevc encoder (codec hevc)
 V..... hevc_qsv             HEVC (Intel Quick Sync Video acceleration) (codec hevc)

## 直接下载已经编译好的shared dll
[x64](https://github.com/BtbN/FFmpeg-Builds/releases)
[x86](https://github.com/sudo-nautilus/FFmpeg-Builds-Win32/releases)

## 编译与安装
[官方帮助文档](https://trac.ffmpeg.org/wiki/CompilationGuide)
### windows下编译
[安装](https://blog.csdn.net/u014552102/article/details/104400885)
1. 下载并安装msys2. [官网](https://www.msys2.org/)
2. 通过开始菜单运行MSYS2 MINGW64，打开命令行窗口。注意：不要运行MSYS2 MSYS。或者命令行运行 C:\\msys2_shell
    > 32位编译运行 msys2_shell.cmd -mingw32
    > 64位编译运行 msys2_shell.cmd -mingw64
3. 在MSYS2 MINGW64内安装依赖
```
pacman -S make
pacman -S diffutils
pacman -S yasm
pacman -S pkg-config
// 如果要编译64位则安装下面这个mingw gcc for x64
pacman -S mingw-w64-x86_64-gcc
// 如果要编译32位则安装下面这个mingw gcc for x86
pacman -S mingw-w64-i686-gcc
```
4. 在MSYS2中导出配置PATH，以便可以使用Windows下安装的git
```
export PATH=$PATH:/c/Program\ Files/Git/cmd
```
5. 下载ffmpeg代码
6. 在MSYS2内进入ffmpeg代码路径，切换到ffmpeg特定版本的tag或者分支
```
cd /f/code/ffmpeg
git checkout n6.0
```
7. 配置，configure
```
// for x64
./configure --enable-shared --enable-decoder=h264 --enable-parser=h264 --arch=x86_64 --prefix=/f/code/ffmpeg/output
// for x86
./configure --enable-shared --enable-decoder=h264 --enable-parser=h264 --arch=x86_32 --prefix=/f/code/ffmpeg/output_x86
```
==很遗憾，使用n6.0版本的tag进行编译，只有x64版本的成功了。x86一直失败==

### linux ubuntu下编译
[官方教程-有效](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu)
```
sudo apt-get update -qq && sudo apt-get -y install \
  autoconf \
  automake \
  build-essential \
  cmake \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  meson \
  ninja-build \
  pkg-config \
  texinfo \
  wget \
  yasm \
  zlib1g-dev
  
  # 当在./configure时enable了某个库，则需要安装对应库的开发包。比如--enable-libfdk-aac对应安装libfdk-aac2 libfdk-aac-dev
  sudo apt install libva2 libva-dev libaom-dev libdavs2-16 libdavs2-dev libfdk-aac2 libfdk-aac-dev libmp3lame0 libmp3lame-dev libopus0 libopus-dev librtmp1 librtmp-dev libsoxr0 libsoxr-dev libspeex1 libspeex-dev libsvtav1enc0 libsvtav1enc-dev libsvtav1dec0 libsvtav1dec-dev libvpx7 libvpx-dev libwebp7 libwebp-dev libunistring-dev
  
  cd ffmpeg
  git checkout release/n5.1
  
./configure --prefix=./output-linux --enable-gpl --enable-nonfree --enable-cuda-nvcc --enable-libvpx --enable-libwebp  --enable-libsvtav1 --enable-libspeex --enable-libsoxr --enable-librtmp --enable-libopus --enable-libmp3lame --enable-libfreetype --enable-libfontconfig --enable-libass --enable-libfdk-aac --enable-libaom   --enable-libdavs2 --enable-cuda --enable-cuvid --enable-nvenc --enable-nvdec --extra-cflags=" -fPIC"
  
  # 如果需要安装支持mediacodec编解码的支持，则需要先安装JNI SDK。然后--enable-jni --enable-mediacodec
  
  # 如果需要支持nvidia cuda编码则configure需要添加参数 --enable-cuda --enable-cuvid --enable-nvenc --enable-nvdec
  # 且在configure之前需要先编译安装nv-codec-headers
  git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
  cd nv-codec-headers
  git checkout <特定分支> 比如安装了cuda 11.8 就切换到11.xx的分支
  make
  sudo make install
  
  
  make -j18
  make install
```

## 编码
### 音频编码AAC
1. 编码AAC时如果使用avcodec_find_encoder_by_name("libfdk_aac");返回nullptr，这是因为编译ffmpeg时没有使用libfdk库。这种情况需要重新编译生成ffmpeg库。在configure时添加以下参数
```
--enable-libfdk-aac --enable-nonfree --enable-gpl
```
2. 如果avcodec用的是libfdk_aac，则在需要将AVCodecContext.sample_fmt设置为AV_SAMPLE_FMT_S16，否则avcodec_open2会返回失败，错误码-22。
3. 如果avcodec用的是avcodec_find_encoder(AV_CODEC_ID_AAC)，则需要将AVCodecContext.sample_fmt设置为AV_SAMPLE_FMT_FLTP，否则avcodec_open2会返回失败，错误码-22。

## 音视频时间戳
先列出以下变量或者常量，之后再理清它们之间的关系。
+ AV_TIME_BASE = 1 000 000   ----  ffmpeg中定义的常量，这个是微秒数，1s==1000000us
+ AVStream.r_frame_rate = AVRational(num / den)  ----  视频AVStream的帧率
+ AVStream.time_base = AVRational(num / den)  ----  视频AVStream的time_base
+ AVCodecContext.time_base 类型为AVRational。用于表示对应音视频数据最小的时间粒度，视频帧公式为：AVCodecContext.time_base = 1 / framerate。音频帧公式为：AVCodecContext.time_base = 1 / samplerate。

### 视频帧的时间戳
+ 每帧的时间增量   duration_ffmpeg = AV_TIME_BASE / frame_rate， 单位为微秒。 
pkt.pts = (double)(frame_index*calc_duration) / (double)(av_q2d(time_base1)*AV_TIME_BASE)

## 项目经验
+ 启动时崩溃
    1. 使用ffmpeg3.3.1版本，编译生成exe一切正常，但是只要一调用ffmpeg程序就崩溃，从Modules的加载来看，没有成功加载ffmpeg的DLL。
设置项目配置-->Linker-->Optimization-->References值为Eliminate Unreferenced Data (/OPT:REF)时将导致崩溃
+ ==linux下编译使用静态库时，链接出错，错误信息如下==
    ```
    libavcodec.a(vc1dsp_mmx.o): relocation R_X86_64_PC32 against symbol `ff_pw_9' can not be used when making a shared object; recompile with -fPIC
    ```
    可能的原因有：

        - 使用的ffmpeg静态库与头文件不是匹配的。
        - 在编译动态库so时链接ffmpeg静态库会报上面的错误，如果直接编译so时不进行链接，而是在构建可执行程序时再链接ffmpeg的静态库就不会报这个链接错误。
+ sws_scale并不是线程安全的。
如果有多个线程使用同一个SwsContext对象调用sws_scale，则可能崩溃。即使处理的是不同的帧也会。如果的确要使用多线程去调用sws_scale，则可以考虑创建多个SwsContext对象， 每个线程使用一个。

## 代码变更
+ 3.x之前版本所用的AVStream.codec已经废弃。[参考](https://blog.51cto.com/fengyuzaitu/1983001)
 
## 命令行命令备忘
解码音频文件并转换成PCM保存
```
// 转换成2 channel 48000采样率  short
 ffmpeg -i src.m4a -acodec pcm_s16le -f s16le -ar 48000 -ac 2 out-ch2-48000-short.pcm
```
解码视频或者图片保存成argb
```
ffmpeg -i input.jpeg -pix_fmt rgba output.tiff
```
将文件夹内的多个图片文件按顺序合并成一个视频文件
```
// 将多个PNG图片按顺序合并成视频文件
// -vf"format=yuva420p”告诉ffmpeg输出WebM视频时使用透明度通道
// -c:v libvpx选择WebM视频编码器
// -auto-alt-refo 禁用自动参考，以降低WebM视频文件大小
// -crf 10 控制视频质量，10为较高的质量，但会增加文件大小
// -b:v 1M 控制视频码率，1M为较高的码率，但会增加文件大小
ffmpeg -framerate 25 -i images/image_%04d.png -vf
"format=yuva420p" -c:v libvpx -auto-alt-ref 0 -crf 10 -b:v 1M output.webm

ffmpeg -framerate 25 -i ./%d.png -vf "format=yuva420p" -c:v libx264 -auto-alt-ref 0 -crf 10 -b:v 4M fore.h264
```
将视频转换为 WebM 并添加透明背景，带抠绿幕
```
// -c:v 选项指定使用 vp9 编码器进行视频编码
// -pix_fmt 选项指定像素格式为 yuva420p，也就是包括 alpha 通道的格式。
// -auto-alt-ref 选项设置参考帧（reference frames）的数量。
// -vf 选项则是指定一个视频过滤器，这里使用了 chromakey 过滤器来实现透明背景：扣掉所有绿色（0x00FF00）的部分，并设置阀值为 0.1（第二个参数）和混合度为 0.2（第三个参数）。
// -b:v 选项指定了视频的码率为 1M，即 1Mbps。
ffmpeg -i input-video.mp4 -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 -vf "chromakey=0x00FF00:0.1:0.2" -b:v 1M output-video.webm
```
编码yuv-->mp4
`ffmpeg.exe -s 2560x720 -f rawvideo -pix_fmt yuv420p -i tmp.yuv420 -r 25 tmp.mp4`
编码yuv+pcm数据成MP4文件
`ffmpeg.exe -f rawvideo -s 1080x1920 -pix_fmt yuv420p -i tmp-1-1080x1920-2.yuv420 -ar 48000 -channels 2 -f s16le -i tmp-1.pcm -r 25 tmp-1.mp4`
使用ffprobo查看所有帧信息
```
 ffprobe.exe -show_frames f25b56750b2694cf2a4b1f3f116e8103.mp3
```
从网络地址中拉流并保存成文件
```
ffmpeg -i https://beauty-tx-hls.meituan.net/mtlr/841895_cd8a7_b6df3974ef2f492_ND1080p.m3u8 -codec copy t.ts
// 保存成mp4, 录制600s
ffmpeg -y -i "rtmp://beauty-tx-rtmp.meituan.net/mtlr/1130991_1141ef_641c29bc00f140c_ND1080p" -vcodec copy -t 600 -f mp4 /d/dump-meituan-linux-server.mp4
```
混画
```
# 将argb数据文件叠加在jpeg的背景上面
ffmpeg -i D:\59b05ab48bf73e56d41e638e8413f699.jpg -f rawvideo -pixel_format rgba -video_size 748x960 -i D:\dump_ai_org-101879.rgb -filter_complex "[1:v]scale=w=748:h=960:force_original_aspect_ratio=decrease[ckout];[0:v][ckout]overlay=x=W-w-10:y=0[out]" -map "[out]" -movflags faststart a.mp4
```
转码--降低分辨率--音频轨道复制
```
ffmpeg -i %%v  -vf scale=720:1280 -preset slow -crf 18 -acodec copy  "720p/%%~nv.mp4"
// bat脚本，批量转换
for /R %%v IN (*.mp4) do ( ffmpeg -i %%v  -vf scale=720:1280 -preset slow -crf 18 -acodec copy  "720p\%%~nv.mp4")
```
批量转换-将jpg转成png
```
find ./ -name  "*.jpg" -type f -exec ffmpeg -i {} {}.png \;
```
使用ffprobe计算视频中的总帧数
```
ffprobe.exe -v error -count_frames -select_streams v:0 -show_entries stream=nb_read_frames -of default=nokey=1:noprint_wrappers=1 fore.h264
```
从a.mp4的第10分钟开始截取60s的数据
```
ffmpeg -ss 00:10:00.000 -t 60 -i a.mp4 a-1.mp4
```
linux捕抓屏幕录制输出到虚拟摄像头v4l2loopback中
```
ffmpeg -f x11grab -select_region 1 -show_region 1 -framerate 25 -i $DISPLAY -vf format=yuv420p -f v4l2 /dev/video0
```
windows将捕抓摄像头+麦克风数据编码成mp4
```
# Windows
ffmpeg -f dshow -i video="Integrated Camera":audio="麦克风阵列 (适用于数字麦克风的英特尔® 智音技术)" -vcodec "h264_nvenc" rec_camera.mp4
# linux v4l2
ffmpeg -f v4l2 -i /dev/video0 -vcodec libx264 ~/record.mp4
```
将普通2D视频转换成左右3D视频
```
# shift 30ms for source video file
ffmpeg -ss 00:00:00.034 -i $1 -vcodec copy -acodec copy tmp_short.mp4

ffmpeg -i $1 -vf "movie=./tmp_short.mp4 [in1]; [in]pad=iw*2:ih:iw:0[in0]; [in0][in1] overlay=0:0 [out]" -vcodec libx264 -preset medium -b:v 9200k -r:v 25 -f mp4 lr-3D.mp4
```
将ass文件的字幕绘制到视频文件上
```
ffmpeg -i video.mp4 -vf ass=subtitle.ass -y dest.mp4
```
将ass文件内容嵌入到视频文件内，由播放器绘制
```
Ffmpeg -i video.mp4 -i subtitle.ass -c:v copy -c:a copy -c:s ass -y dest.mkv
```
摄像头相关-linux
```
# 打开摄像头播放 -framerate 30 -video_size hd720
ffplay -f video4linux2 /dev/video0
# 打印摄像头支持的格式
ffplay -f video4linux2 -list_formats all /dev/video0

# 使用ffplay查看yuv文件
//  -x 1280 -y 360 可指定显示的窗口大小
ffplay -f rawvideo -pixel_format nv21 -video_size 1280x960 testCame_1280x960_2.yuv

# 是用ffplay播放PCM文件
ffplay -ar 44100 -channels 2 -f s16le -i ~/Downloads/capture_audio.pcm
```
#### 特效命令行
[zoompan的使用例子](https://blog.csdn.net/linzhiji/article/details/133020028)
```
# 分辨率500x500的延时视频放大到1080x1920，原始视频宽放大到铺满1080，高不够1920的部分进行填充
ffmpeg -i E:\output.mp4 -vf "scale=1080:1080:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:color=black" E:\output-2.mp4

# 先将500x500图片放大到1080x1920，再使用zoompan进行镜头推进效果的生成
ffmpeg -loop 1 -i D:\data_for_test\vinda_1.jpg -vf "scale=1080:1080, pad=1080:1920:0:(oh-ih)/2,zoompan= z='min(zoom+0.002,1.3)':x='iw/2-(iw/zoom/2)':y='(1920/2-(1080/zoom/2))':d=125:s=1080x1920" -t 2 -r 25 -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p E:\output-3.mp4
ffmpeg -loop 1 -i D:\data_for_test\vinda_1.jpg -vf "
    scale=1080:1080,                # 先缩放到1080x1080正方形
    pad=1080:1920:0:(oh-ih)/2,      # 上下填充黑边变成1080x1920
    zoompan=
        z='min(zoom+0.002,1.3)':    # 放大系数从1.0到1.3倍
        x='iw/2-(iw/zoom/2)':       # 保持X轴居中
        y='(1920/2-(1080/zoom/2))': # 计算竖屏Y轴居中
        d=125:                      # 持续125帧（25fps时为5秒）
        s=1080x1920"                # 输出分辨率
    -t 2 -r 25                      # 5秒时长，25帧率
    -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p 
    E:\output-3.mp4

# 未测试。使用原始图片的边缘颜色进行填充。失败信息: No such filter: 'gegl'
ffmpeg -loop 1 -i D:\data_for_test\vinda_1.jpg -vf "scale=1080:1080, pad=1080:1920:0:(oh-ih)/2:color=#000000@0,crop=500:500:0:0,gegl='operation=gegl:edge-extend mode=smear', scale=1080:1080,pad=1080:1920:0:(oh-ih)/2,zoompan= z='min(zoom+0.002,1.3)':x='iw/2-(iw/zoom/2)':y='(1920/2-(1080/zoom/2))':d=125:s=1080x1920" -t 2 -r 25 -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p E:\output-3.mp4
```

### ffmpeg读文件解码的特性
1. 先读到文件的末尾，此时已经产生了EOF的错误，但是解码其还是正常解码。on