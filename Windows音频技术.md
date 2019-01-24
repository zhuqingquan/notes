# Windows音频技术
## DirectShow技术
DirectShow包含了音视频的内容，是音视频处理的框架。DirectShow会使用DirectSound等技术来管理设备，完成音频数据采集，音频播放等。DirectShow是面向程序应用开发的音视频处理开发框架。
![音频架构图](https://raw.githubusercontent.com/zhuqingquan/notes/master/resource/DirectShow-arch-oview2.png)

## DirectSound
[微软doc链接](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416960(v%3dvs.85))
### 音频输入采集
#### 获取所有音频采集设备
* 相关函数[DirectSoundCaptureEnumerate](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416761%28v%3dvs.85%29)、[DSEnumCallback](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416829%28v%3dvs.85%29)
* **DirectSound的这个接口并不能枚举获取到所有的音频输入设备，对于某些采集卡内置声道无法获取到**
* DSEnumCallbak只能获取到设备的GUID、描述信息（设备显示的名称）、路径id。而且描述信息只有32个字符，可能无法显示完整的设备名称。

#### 创建Capture、Buffer
* [介绍的链接](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416362%28v%3dvs.85%29)
* 相关函数 [DirectSoundCaptureCreate](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416760%28v%3dvs.85%29)、[IDirectSoundCapture8::CreateCaptureBuffer](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee416355(v%3dvs.85))、[WAVEFORMATEX](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee419019%28v%3dvs.85%29)
* 在创建Capture Buffer时需要传入WAVEFORMATEX结构体，并根据WAVEFORMATEX计算一个能够以sample对齐的buffer长度。成功创建Buffer之后，Buffer中的数据就是按照WAVEFORMATEX制定的格式保存。
```
HRESULT CreateCaptureBuffer(LPDIRECTSOUNDCAPTURE8 pDSC, 
                            LPDIRECTSOUNDCAPTUREBUFFER8* ppDSCB8)
{
  HRESULT hr;
  DSCBUFFERDESC               dscbd;
  LPDIRECTSOUNDCAPTUREBUFFER  pDSCB;
  WAVEFORMATEX                wfx =
    {WAVE_FORMAT_PCM, 2, 44100, 176400, 4, 16, 0};
    // wFormatTag, nChannels, nSamplesPerSec, mAvgBytesPerSec,
    // nBlockAlign, wBitsPerSample, cbSize
 
  if ((NULL == pDSC) || (NULL == ppDSCB8)) return E_INVALIDARG;
  dscbd.dwSize = sizeof(DSCBUFFERDESC);
  dscbd.dwFlags = 0;
  dscbd.dwBufferBytes = wfx.nAvgBytesPerSec;
  dscbd.dwReserved = 0;
  dscbd.lpwfxFormat = &wfx;
  dscbd.dwFXCount = 0;
  dscbd.lpDSCFXDesc = NULL;
 
  if (SUCCEEDED(hr = pDSC->CreateCaptureBuffer(&dscbd, &pDSCB, NULL)))
  {
    hr = pDSCB->QueryInterface(IID_IDirectSoundCaptureBuffer8, (LPVOID*)ppDSCB8);
    pDSCB->Release();  
  }
  return hr;
}
```
#### 从Capture Buffer中读取数据
* [介绍的链接](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee419015(v%3dvs.85))
* 相关函数 [IDirectSoundCaptureBuffer8::GetCurrentPosition](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee418166%28v%3dvs.85%29)、[IDirectSoundCaptureBuffer8::Lock](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee418179%28v%3dvs.85%29)、[IDirectSoundCaptureBuffer8::Unlock](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee418185%28v%3dvs.85%29)、[IDirectSoundNotify8::SetNotificationPositions](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee418245%28v%3dvs.85%29)
* 通过调用**Lock**函数获取Capture Buffer的内存地址，然后进行数据拷贝，拷贝完成后必须进行**Unlock**。
* Capture Buffer是一个循环Buffer。调用者保存上一次拷贝完成后的数据末尾的偏移量*rpos1*，之后每次读取之前调用**GetCurrentPosition**获取当前可读取的数据的末尾的偏移量*rpos2*，[*rpos2, rpos1*]区间内的数据就是需要拷贝的数据。**注意因为Capture Buffer是循环Buffer，因此可能rpos2 < rpos1**。
* 读取Capture Buffer的时机：
> * 使用循环不断调用**GetCurrentPosition**，**Lock**，**Unlock**读取Buffer中的值
> * 使用**SetNotificationPositions**设置事件（Event）触发的读取点。当Capture Buffer在某个Position可读时触发事件，此时再去读取。
___
## WDM Audio
[微软doc链接](https://docs.microsoft.com/en-us/windows-hardware/drivers/audio/wdm-audio-architecture--basic-concepts)
### Kernel Streaming
A KS filter is implemented as a kernel-mode driver object that encapsulates some number of related stream-processing functions. The functionality can be implemented in software or in hardware. In this model, an audio adapter can be viewed as a collection of hardware devices, and the adapter driver exposes each of these devices to the audio system as an individual filter.
* adapter 显卡硬件设备
* device 显卡中包含的逻辑设备
* 一个adapter可以包含多个device，一般每个device会有一个filter与之对应。

___
## DirectMusic
## Windows multimedia waveXxx and mixerXxx functions
## Media Foundation
## 关于Kernel Streaming
[官网链接](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/stream/kernel-streaming)

## Core Audio API
[微软主页 Core Audio APIs](https://docs.microsoft.com/zh-cn/windows/desktop/CoreAudio/about-the-windows-core-audio-apis)
### Multimedia Device (MMDevice) API
### Windows Audio Session API (WASAPI)
### DeviceTopology API
### EndpointVolume API