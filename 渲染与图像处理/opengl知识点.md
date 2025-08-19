## 辅助库
* **glfw** 创建窗口、显示器管理API、设置窗口属性（Hints)、管理context、窗口事件管理、鼠标键盘事件管理、遥控杆支持
* **glew** 跨平台，取代了原来的gl.h、wgl.h，让你可以方便的调用最新的OpenGL功能，包括OpenGL众多的Extension。
* **glm**
* **glut**

## Context与资源初始化
1. **glfwInit()** 对应在程序结束时使用 glfwTerminate()。整个进程调用一次glfwInit()。
2. **glfwWindowHint()** 设置需要创建的窗口的属性
3. **glfwCreateWindow()** 创建指定大小的window与和这个window关联的context。
4. **glfwMakeContextCurrent(window)** 设置当前的context
5. **glewInit()** 初始化glew库
6. **glClearColor(R,G,B,A)** 

## 顶点数据Buffer

#### 顶点数组对象 vertex-array object (VAO)
> **VAO指定了读取VBO的方式** <br>
> 通过调用**glGenVertexArrays**从opengl中申请N个顶点数组对象。<br>
> 调用**glBindVertexArray**激活顶点数组对象 <br>
> 关于VAO作用的解释 https://www.zhihu.com/question/30095978 <br>

```
GLuint VertexArrayID;
glGenVertexArrays(1, &VertexArrayID);
glBindVertexArray(VertexArrayID);

glDeleteVertexArrays(1, &VertexArrayID);
```
### 顶点数据对象 vertex-buffer object (VBO)
> 保存顶点信息的数据，如(x,y,z,px,py) <br>
> 顶点数据buffer需要绑定为**GL_ARRAY_BUFFER**类型 <br>

```
	static const GLfloat g_vertex_buffer_data[] = { 
		-1.0f, -1.0f, 0.0f,
		 1.0f, -1.0f, 0.0f,
		 0.0f,  1.0f, 0.0f,
	};

	GLuint vertexbuffer;
	glGenBuffers(1, &vertexbuffer);
	glBindBuffer(GL_ARRAY_BUFFER, vertexbuffer);
	glBufferData(GL_ARRAY_BUFFER, sizeof(g_vertex_buffer_data), g_vertex_buffer_data, GL_STATIC_DRAW);
	
	// Cleanup VBO
	glDeleteBuffers(1, &vertexbuffer);
```  
### 顶点数据格式描述，关联Vertex Shader
> 描述顶点数据的格式，并将数据关联绑定到Vertex Shader中的in变量中。<br> 
> 调用**glVertexAttribPointer**()实现 <br>
> glVertexAttribPointer()参数中index表明当前绑定的vertex buffer与Vertex Shader中哪个location值的in变量关联。<br>
> **glEnableVertexAttribArray与glVertexAttribPointer参数index使用同一个数值** <br>
> **使用glEnableVeretexAttribArray与glVertexAttribPointer设置的状态保存在当前绑定的顶点数组对象（VAO)中。**

```
    // 1rst attribute buffer : vertices
	glEnableVertexAttribArray(0);
	glBindBuffer(GL_ARRAY_BUFFER, vertexbuffer);
	glVertexAttribPointer(
		0,                  // attribute 0. No particular reason for 0, but must match the layout in the shader.
		3,                  // size
		GL_FLOAT,           // type
		GL_FALSE,           // normalized?
		0,                  // stride
		(void*)0            // array buffer offset
	);
	
	glDisableVertexAttribArray(0);
```
___
## 纹理Texture相关
[wiki-Texture Storage](https://www.khronos.org/opengl/wiki/Texture_Storage)
[wiki-Pixel Transfer](https://www.khronos.org/opengl/wiki/Pixel_Transfer)
### 纹理Texture的创建与初始化
> 使用**glGenTextures或者glCreateTextures**申请空闲的Texture <br>
> 为Texture分配内存 **glTextureStorage2D**。一旦我们为纹理分配了存储空间，那么它就无法被重新分配或者释放，只在纹理释放时才会释放对应的空间。

```
	int pic_w = 1920;
	int pic_h = 1080;
	// create texture 2d
	GLuint texture = 0;
	glGenTextures(1, &texture);
	glBindTexture(GL_TEXTURE_2D, texture);
	glTextureStorage2D(texture, 1, GL_RGBA8, pic_w, pic_h);
```

### 更新纹理数据 glTextureSubImagexxx
> **glTextureSubImage2D**实现将内存中的数据更新到Texture中。<br>
> 参数**format**支持**GL_RED、GL_RGBA等** <br>
> 参数**type**支持**GL_UNSIGNED_BYTE、GL_FLOAT等** <br>
> 参数**pixels**可以是用户的内存指针，也可以是GL_PIXEL_UNPACK_BUFFER target的缓存对象的偏移量。**如果GL_PIXEL_UNPACK_BUFFER没有绑定任何缓存对象，则pixels解释为本地指针，否则则为缓存对象的偏移量。

##### 从内存中拷贝内容到纹理Texture中
``` 
    // 直接从内存拷贝数据的纹理Texture中
    uint32 pic_len = pic_w * 4 * pic_h;
	uint8_t* picBuffer = (uint8_t*)malloc(pic_len);
	memset(picBuffer, 0x66, pic_len);
	glTextureSubImage2D(texture, 0, 0, 0, pic_w, pic_h, GL_RGBA, GL_UNSIGNED_BYTE, picBuffer);
```
##### 从缓存对象GL_PIXEL_UNPACK_BUFFER中拷贝数据到纹理Texture中
[关于PBO的用法](https://zhuanlan.zhihu.com/p/115257287)
+ glBufferData与glNamedBufferStorage是一样的效果，只不过glBufferData调用之前需要先binding buffer。
+ glTextureSubImage2D与glTexImage2D的区别：==glTexImage2D在Texture没有分配显存资源时会触发分配显存资源，然后再进行初始化或者数据拷贝。而glTextureSubImage2D不会分配显存资源，如果没有调用过glTexImage2D而直接调用glTextureSubImage2D会因为没有分配资源而出现黑屏之类的问题==。
+ 注意：==代码中我们调用glBufferData时usage传的是GL_STREAM_DRAW==。
```
	// 先创建缓存buffer，绑定GL_PIXEL_UNPACK_BUFFER，然后再从缓存Buffer中拷贝到纹理Texture中
	GLuint picCacheBuffer;
	glCreateBuffers(1, &picCacheBuffer);
	{
	glNamedBufferStorage(picCacheBuffer, pic_len, picBuffer, 0);
	// 或者
	glBindBuffer(GL_PIXEL_UNPACK_BUFFER, buffer);
	glBufferData(GL_PIXEL_UNPACK_BUFFER, stride * h, nullptr, GL_STREAM_DRAW);
	}
	// 将缓存Buffer绑定到GL_PIXEL_UNPACK_BUFFER
	glBindBuffer(GL_PIXEL_UNPACK_BUFFER, picCacheBuffer);
	glTextureSubImage2D(texture, 0, 0, 0, pic_w, pic_h, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
```

### 纹理Texture启用，绑定FragmentShader
> 通过**glGetUniformLocation**()获取需要绑定的FragmentShader中声明的sampler2D对象。<br>
> 调用 **glActiveTexture**()启用GL_TEXTURE0序号的Texture <br>
> 绑定创建的Texture到GL_TEXTURE0位置上**glBindTexture**() <br>
> **glBindTextureUnit**可以将Texture绑定到指定的纹理单元(Texture Unit)中,此时Texture需要通过glCreateTexture创建指定了维度（2D...)或者使用glBindTexture指定了Texture。
> **glUniform1i**将GL_TEXTURE0位置上的Texture与FragmentShader中的sampler2D绑定。<br>
> 关于启用Texture中调用**glActiveTexture与glBindTexture**不同点的说明，https://blog.csdn.net/artisans/article/details/76695614

```
    GLuint Texture = CreateTexture();   //create texture
	// Get a handle for our "myTextureSampler" uniform
	GLuint TextureID  = glGetUniformLocation(programID, "myTextureSampler");
	
	// Bind our texture in Texture Unit 0
	glActiveTexture(GL_TEXTURE0);
	glBindTexture(GL_TEXTURE_2D, Texture);
	// Set our "myTextureSampler" sampler to use Texture Unit 0
	glUniform1i(TextureID, 0);
```

### 拷贝纹理Texture数据到内存中
> 使用**glGetTextureImage**将Texture数据拷贝到内存或者缓冲buffer对象中<br>
> 直接将纹理Texture数据拷贝到内存中效率不高。建议使用GL_PIXEL_PACK_BUFFER绑定缓存对象的方式，先将像素数据拷贝到缓存Buffer对象中，之后再把缓存Buffer对象映射到内存中提供给应用层读取。

```
void glGetTextureImage(GLuint texture, GLint level, GLenum format, GLenum type, GLsizei bufSize, void* pixels);
```

### 从FrameBuffer拷贝到Texture
[文档参考](https://www.khronos.org/opengl/wiki/Texture_Storage#Framebuffer_copy_creation)
```
// 需要先绑定FrameBuffer，确保FrameBuffer是complete
// 将RTT的Texture内容拷贝到独立的Texture中，之后使用此Texture进行回调
    glReadBuffer(GL_FRONT);
    glBindTexture(m_textureRTTDumped->glTexTarget(), m_textureRTTDumped->glTextureID());
    glCopyTexImage2D(m_textureRTTDumped->glTexTarget(), 0, m_textureRTTDumped->glTexInternalFmt(), 0, 0, m_RTTWidth, m_RTTHeight, 0);
    glBindTexture(m_textureRTTDumped->glTexTarget(), 0);
```
___

### 共享纹理等资源Shared texture、shared context
[linux - 如何在 2 个进程(linux)之间共享 OpenGL 上下文/纹理](https://www.coder.work/article/7496411)

___
### 离屏渲染（Offscreen Rendering）
> 将画面渲染到纹理Texture中 <br>
> 步骤
>> 使用**glCreateFramebuffer**获取Frame buffer对象。获取的Frame buffer对象可通过**glDeleteFramebuffers**释放且解除绑定。 <br>
>> 使用**glBindFramebuffer**将获取的Frame buffer对象绑定到当前环境中。绑定后，Frame buffer对象将处于激活状态，任何后继的OpenGL渲染操作都会被绘制到这个Frame Buffer对象中。<br>
>> 使用**glNamedFramebufferTexture**将纹理绑定到Framebuffer上作为Framebuffer的附件(Framebuffer attachment) <br>
>> 绘制完成后如果需要重新启用窗口默认的Framebuffer，则需要调用**glBindFramebuffer(GL_FRAMEBUFFER, 0)**  <br>
> **注意：glBindFramebuffer(GL_READ_FRAMEBUFFER, offsceen)与glBlitFramebuffer**

##### 渲染到渲染缓存(Renderbuffer)
> Renderbuffer是OpenGL管理的一处高效的**内存区域**，可以存储格式化的图像数据。<br>
> 可以像绑定Texture到Framebuffer一样绑定Renderbuffer到Framebuffer上，从而实现直接渲染到Renderbuffer中。<br>
> 基本操作函数
>> glCreateRenderbuffers()  
>> glDeleteRenderbuffers()  
>> glBindRenderbuffer(GL_RENDERBUFFER, rb) <br>
> 在将Renderbuffer关联到Framebuffer中渲染之前，需要分配存储空间并设置图像格式。**glNamedRenderbufferStorage**  
> 使用**glNamedFramebufferRenderbuffer**将Renderbuffer关联到Framebuffer的Attachment上。  


```
	// 创建Offscreen texture与Framebuffer
	int dst_w = 1920;
	int dst_h = 1080;
	// create texture2D
	GLuint offscreen_texture;
	glCreateTextures(GL_TEXTURE_2D, 1, &offscreen_texture);
	glTextureStorage2D(offscreen_texture, 0, GL_RGBA8, dst_w, dst_h);
	// create Framebuffer
	GLuint framebuf;
	glCreateFramebuffers(1, &framebuf);
	// bind texture to Framebuffer
	glNamedFramebufferTexture1DEXT(framebuf, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, offscreen_texture, 0);
	
	// 渲染到Offscreen texture上
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, framebuf);
	// viewport设置成与texture大小一致
	glViewport(0, 0, dst_w, dst_h);
	
	// 省略绘制代码
	
	// 重新将窗口默认Framebuffer激活
	glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

### Texture相关的数据拷贝
##### 从内存中拷贝数据到Texture

```
graph LR
内存-->Texture
```
```
```

##### Texture之间相互拷贝
```
graph LR
Texture-A-->Texture-B
```
##### 从Texture拷贝到内存
```
graph LR
Texture-->内存
```

___
### 透明渲染
1. 如果Texture的像素是ARGB之类的带alpha通道的。则可以直接打开Blend并设置Blend function即可。
```
    if (m_enableTransparent)
	{
		// 开启
		glEnable(GL_BLEND);
		glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
	}
    int ret = doDraw();
	if (m_enableTransparent)
	{
		glDisable(GL_BLEND);
	}
```
2. 离屏渲染offscreen到PBO中支持透明背景
[windows下的参考](https://stackoverflow.com/questions/30015597/how-can-i-create-opengl-context-using-windows-memory-dc-c)
+ Windows下有一种方式是创建一个DIBSection、CreateCompatibleDC作为HDC用于创建gl Context。Context可以创建成功，但是在这个context下只能使用OpenGL 1.1版本的API。以下记录以下代码：
```
HDC createDIB(int cx, int cy, HBITMAP& hbmpDIB)
{
    assert(cx > 0);
    assert(cy > 0);

    int cxDIB(cx);
    int cyDIB(cy);

    BITMAPINFOHEADER BIH;
    int iSize = sizeof(BITMAPINFOHEADER);
    memset(&BIH, 0, iSize);
    BIH.biSize = iSize;
    BIH.biWidth = cx;
    BIH.biHeight = cy;
    BIH.biPlanes = 1;
    BIH.biBitCount = 32;
    BIH.biCompression = BI_RGB;

    HDC pdcDIB = CreateCompatibleDC(NULL);
    assert(pdcDIB);
    if (NULL == pdcDIB)
    {
        loge("CreateCompatibleDC failed.error=%d", GetLastError());
        return NULL;
    }

    void* bmp_cnt(NULL);
    HBITMAP bmpDIB = CreateDIBSection(
        pdcDIB,
        (BITMAPINFO*)&BIH,
        DIB_RGB_COLORS,
        &bmp_cnt,
        NULL,
        0);

    assert(bmpDIB);
    assert(bmp_cnt);
    if (bmpDIB == NULL)
    {
        loge("CreateDIBSection failed.error=%d", GetLastError());
        DeleteDC(pdcDIB);
        return NULL;
    }

    hbmpDIB = bmpDIB;
    SelectObject(pdcDIB, hbmpDIB);
    return pdcDIB;
}

BOOL setPixelFormatDIB(HDC pdcDIB)
{
    DWORD dwFlags = PFD_SUPPORT_OPENGL | PFD_DRAW_TO_BITMAP;

    PIXELFORMATDESCRIPTOR pfd;
    memset(&pfd, 0, sizeof(PIXELFORMATDESCRIPTOR));
    pfd.nSize = sizeof(PIXELFORMATDESCRIPTOR);
    pfd.nVersion = 1;
    pfd.dwFlags = dwFlags;
    pfd.iPixelType = PFD_TYPE_RGBA;
    pfd.cColorBits = 32;
    pfd.cDepthBits = 16;
    pfd.iLayerType = 0;

    int PixelFormat = ChoosePixelFormat(pdcDIB, &pfd);
    if (PixelFormat == 0) {
        assert(0);
        DWORD errorCode = GetLastError();
        loge("ChoosePixelFormat ErrorCode: %u", errorCode);
        return FALSE;
    }

    BOOL bResult = SetPixelFormat(pdcDIB, PixelFormat, &pfd);
    if (bResult == FALSE) {
        DWORD errorCode = GetLastError();
        loge("SetPixelFormat ErrorCode: %u", errorCode);
        return false;
        //assert(0);
        //return FALSE;
    }

    return TRUE;
}
```
3. Windows下创建Layed Window，基于此Window的HDC创建的PBO可以支持透明通道。
```
bool WinCreate(HWND& _hWindow)
{
	WNDCLASSW wndclass = { 0 };
	DWORD    wStyle = 0;
	RECT     windowRect;
	HINSTANCE hInstance = GetModuleHandle(NULL);

	wndclass.style = CS_OWNDC;
	wndclass.lpfnWndProc = (WNDPROC)ESWindowProc;
	wndclass.hInstance = hInstance;
	wndclass.hbrBackground = (HBRUSH)COLOR_BACKGROUND;//(HBRUSH)GetStockObject(BLACK_BRUSH);

	wchar_t* w_className = L"GLUtils-class";
	wndclass.lpszClassName = w_className;

	if (!RegisterClassW(&wndclass))
	{
		int err = GetLastError();
		if(err!=ERROR_CLASS_ALREADY_EXISTS)
		{
			loge("RegisterClassW failed. ERR= %d", err);
			return FALSE;
		}
	}

	wStyle = WS_OVERLAPPEDWINDOW | WS_CLIPSIBLINGS | WS_CLIPCHILDREN;

	// Adjust the window rectangle so that the client area has
	// the correct number of pixels
	windowRect.left = 0;
	windowRect.top = 0;
	windowRect.right = CW_USEDEFAULT;
	windowRect.bottom = CW_USEDEFAULT;

	AdjustWindowRect(&windowRect, wStyle, FALSE);

	//_hWindow = CreateWindowW(
	//	w_className,
	//	L"ZMEDIA-GLUTILS",
	//	wStyle,
	//	0,
	//	0,
	//	windowRect.right - windowRect.left,
	//	windowRect.bottom - windowRect.top,
	//	NULL,
	//	NULL,
	//	hInstance,
	//	NULL);
    
    // 此处使用CreateWindowEx并使用WS_EX_LAYERED的style，这样子绘制的PBO才能支持透明通道
    _hWindow = CreateWindowExW(
        WS_EX_LAYERED,
        w_className,
        L"ZMEDIA-GLUTILS",
        wStyle,
        0,
        0,
        windowRect.right - windowRect.left,
        windowRect.bottom - windowRect.top,
        NULL,
        NULL,
        hInstance,
        NULL);

	//ShowWindow(_hWindow, SW_HIDE);

	if (_hWindow == NULL)
	{
		return false;
	}
	return true;
}
```
4. 需要渲染时开启透明则需要在设置glEnable(GL_BLEND)
```
    glEnable(GL_ALPHA_TEST);
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```
5. 参考的链接中还提到一种使用WGL拓展的方式，没有尝试。

___
### 启用纹理mipmap
1. 启用方式
```
glBindTexture(tex.texture->glTexTarget(), tex.texture->glTextureID());
glTexImage2D(tex.texture->glTexTarget(), 0, DEFAULT_TEXTURE_INTERNAL_FORMAT, fmt.w, fmt.h, 0, pixfmt, GL_UNSIGNED_BYTE, nullptr);
// copy data to pbo
int glStride = fmt.w * 4;
GLuint buf = tex.texture->glBuffer();
GLUtils::updatePBOBufferAndCopy(buf, glStride, fmt.w, fmt.h, pic->rgb(), fmt.stride, tex.texture->glTexTarget(), DEFAULT_TEXTURE_INTERNAL_FORMAT);
glGenerateMipmap(tex.texture->glTexTarget());
```
+ 需要先创建Texture，并为Texture分配显存空间，然后初始化Texture内容或者通过PBO去初始化。
+ 然后再调用glGenerateMipmap进行生成mipmap各个级别的Texture
+ 每次更新了Texture之后，都需要调用glGenerateMipmap进行相应的更新操作。

2. 优缺点
加入mipmap，在渲染曲线时，曲线将变得更加平滑，字体也会更平滑。缺点是：整个画面的对比度看起来下降了，看起来效果是画面感觉更模糊了。

### 抗锯齿
#### MSAA
[LearnOpenGL-抗锯齿](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/11%20Anti%20Aliasing/)
[anti_aliasing_offscreen.cpp](https://learnopengl.com/code_viewer_gh.php?code=src/4.advanced_opengl/11.2.anti_aliasing_offscreen/anti_aliasing_offscreen.cpp)
___
## 着色器Shader
### uniform变量相关
* uniform变量值由应用程序设置，shader只能读取，不能写入、修改。
* uniform变量在所有可用的shader阶段之间是共享的。
* GLSL编译器会在链接shader程序时创建一个uniform变量列表
#### 如何设置uniform变量的值
* 调用**glGetUniformLocation**获取变量在unifor变量列表中的索引
* 使用获取到的索引值调用**glUniformxxx或者glUniformMatrixxxx**设置变量值   
* 如果uniform变量在shader只声明而没有使用，那么glGetUniformLocation返回值将为-1;

API | 用途
---|---
glUniform{1234}{fdi ui} | 设置单个值，向量变量的值
glUniform{1234}{fdi ui}v | 设置数组的值
glUniforMatrix{1234}{fd}v | 设置2x2,3x3,4x4矩阵的值
glUniforMatrix{nxm}{fd}v | 设置非2x2,3x3,4x4的矩阵的值

___
## 其他零散知识
### glGenxxx与glCreatexxx的区别
> 都是创建申请资源，但是一般glGenxxx调用只是申请占用了预留的名字，但这个名字对应的资源类型还未绑定，所以一般还需要再调用glBindxxx进行类型绑定。而glCreatexxx则可以一次性完成资源申请与类型初始化绑定。

### 关于glewExperimental
> 请注意，我们在初始化GLEW之前设置**glewExperimental**变量的值为GL_TRUE，这样做能让GLEW在管理OpenGL的函数指针时更多地使用现代化的技术，如果把它设置为GL_FALSE的话可能会在使用OpenGL的核心模式时出现一些问题。<br>

### 关于Framebuffer的资料
[Framebuffer相关知识](https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/05%20Framebuffers/)   
[优秀的解释说明以及代码实例](https://blog.csdn.net/traceorigin/article/details/9079521)  
[貌似稍微深入一点](https://blog.csdn.net/hgl868/article/details/40055279)  
FBO是一个容器，他有三个attachment点：颜色缓存，深度缓存，和模板缓存的attachment 点。还保存着，添加到这些attachment点上的颜色缓存，深度缓存，模板缓存的大小、格式等属性，以及添加进来的纹理和rbo的名字等等。
> 颜色缓存attachment点，可以添加纹理或rbo。  
> 深度缓存attachment点，可以添加纹理或rbo。   
> 模板缓存attachment点，只能添加rbo。  
**添加到一个fbo中attachment点的各个rbo或纹理才尺寸必须一致。**

### 性能优化点
> 使用glInvalidataNamedFramebufferData替换glClear
>> glInvalidateNamedFramebufferData、glInvalidateNamedFramebufferSubData
>> glInvalidateTexImage、glInvalidateTexSubImage  

> 关于FBO、RBO的使用对性能的影响
>> 频繁的在自己创建的fbo和窗口系统创建的vbo之间切换，比较影响性能。  
>> 不要在每一帧都去创建，销毁fbo，vbo对象。要一次创建多次使用。  
>> 如果一个纹理attach到一个fbo的attachment point，就要尽量避免调用glTexImage2D或glTexSubImage2D      glCopyTexImage2D等去修改纹理的值。

___
### 项目经验
#### 关于崩溃
* 如果两个dll都使用了glew静态库，当两个项目中的代码都会在同一个线程中调用glew的初始化时会导致崩溃。（非同一线程未测试验证是否崩溃）
* 如果两个dll都使用了glew动态链接库，而glew的版本不同，此时glxxx函数调用可能在某个函数中会崩溃。

#### linux server中运行OpenGL渲染
需要安装xvfb虚拟屏幕，这样才能够顺利初始化EGL和OpenGL环境。

#### Opengl中Texture的UV坐标系
Opengl中Texture在Filte时UV坐标系，**点(0,0)代表的是图片的左下角**。在DX中点(0,0)代表的是右上角。这个差别需要注意。

#### 关于GL_INVALID_OPERATION 0x0502(1282)
有时调用一些普通的api，如果glDeleteFrameBuffer之类的都会失败，并且通过glGetLastError()返回的错误码是，0x502(1282)，即GL_INVALID_OPERATION。此时大概率是因为Current context未设置。

#### WSL中使用opengl
wsl安装后，安装Ubuntu系统，此时OpenGL的mesa环境并未安装。需要手动安装，不然在 eglGetDisplay(EGL_DEFAULT_DISPLAY)时会失败。
```
# 安装mesa-utils
sudo apt install mesa-utils
# 确认OpenGL运行环境OK
glxinfo | grep OpenGL
```

#### libglvnd
[代码路径](https://gitlab.freedesktop.org/glvnd/libglvnd)
很重要的一个库，对于OpenGL的环境部署的理解有很大帮助。可以看看这个库的readme帮助文档。

这个库实现了将EGL、OpenGL的API调用分发（调度）到特定的vendor（比如NVIDIA或者MESA 3D）实现的库对于的API上。

#### GBM（Generic Buffer Manager）
##### 有什么用？？？
> Mesa GBM (Generic Buffer Manager) basically provides a EGL native window type (just like Wayland and X11), so one could obtain a real EGL surface and create render target buffers. With that then, GL can be used to render into these buffers, which will be shown to the display by queuing a page flip via KMS/DRM API.

GBM可以提供初始化EGL所需的surface并用于创建Render Target，类似Waylan或者x11中的native Window。这样可以作为Wayland, X11的替代？当OpenGL渲染内容到这个EGL surface后，可以使用DRM的API直接显示到显示器上？

> EGL/GBM does not support Pbuffers. It doesn't support Pixmaps either. To create an offscreen surface with EGL/GBM, you must pass a gbm_surface to eglCreateWindowSurface. "Window" is a misnomer. No real "window" gets created. The resultant buffer will remain offscreen unless you use the kernel's KMS APIs to post it to the display.

##### GBM + DRM的代码例子demo
```
// gcc -o drm-gbm drm-gbm.c -ldrm -lgbm -lEGL -lGL -I/usr/include/libdrm

// general documentation: man drm

#include <xf86drm.h>
#include <xf86drmMode.h>
#include <gbm.h>
#include <EGL/egl.h>
#include <GL/gl.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

#define EXIT(msg) { fputs (msg, stderr); exit (EXIT_FAILURE); }

static int device;

static drmModeConnector *find_connector (drmModeRes *resources) {
	// iterate the connectors
	int i;
	for (i=0; i<resources->count_connectors; i++) {
		drmModeConnector *connector = drmModeGetConnector (device, resources->connectors[i]);
		// pick the first connected connector
		if (connector->connection == DRM_MODE_CONNECTED) {
			return connector;
		}
		drmModeFreeConnector (connector);
	}
	// no connector found
	return NULL;
}

static drmModeEncoder *find_encoder (drmModeRes *resources, drmModeConnector *connector) {
	if (connector->encoder_id) {
		return drmModeGetEncoder (device, connector->encoder_id);
	}
	// no encoder found
	return NULL;
}

static uint32_t connector_id;
static drmModeModeInfo mode_info;
static drmModeCrtc *crtc;

static void find_display_configuration () {
	drmModeRes *resources = drmModeGetResources (device);
	// find a connector
	drmModeConnector *connector = find_connector (resources);
	if (!connector) EXIT ("no connector found\n");
	// save the connector_id
	connector_id = connector->connector_id;
	// save the first mode
	mode_info = connector->modes[0];
	printf ("resolution: %ix%i\n", mode_info.hdisplay, mode_info.vdisplay);
	// find an encoder
	drmModeEncoder *encoder = find_encoder (resources, connector);
	if (!encoder) EXIT ("no encoder found\n");
	// find a CRTC
	if (encoder->crtc_id) {
		crtc = drmModeGetCrtc (device, encoder->crtc_id);
	}
	drmModeFreeEncoder (encoder);
	drmModeFreeConnector (connector);
	drmModeFreeResources (resources);
}

static struct gbm_device *gbm_device;
static EGLDisplay display;
static EGLContext context;
static struct gbm_surface *gbm_surface;
static EGLSurface egl_surface;

static void setup_opengl () {
	gbm_device = gbm_create_device (device);
	display = eglGetDisplay (gbm_device);
	eglInitialize (display, NULL, NULL);
	
	// create an OpenGL context
	eglBindAPI (EGL_OPENGL_API);
	EGLint attributes[] = {
		EGL_RED_SIZE, 8,
		EGL_GREEN_SIZE, 8,
		EGL_BLUE_SIZE, 8,
	EGL_NONE};
	EGLConfig config;
	EGLint num_config;
	eglChooseConfig (display, attributes, &config, 1, &num_config);
	context = eglCreateContext (display, config, EGL_NO_CONTEXT, NULL);
	
	// create the GBM and EGL surface
	gbm_surface = gbm_surface_create (gbm_device, mode_info.hdisplay, mode_info.vdisplay, GBM_BO_FORMAT_XRGB8888, GBM_BO_USE_SCANOUT|GBM_BO_USE_RENDERING);
	egl_surface = eglCreateWindowSurface (display, config, gbm_surface, NULL);
	eglMakeCurrent (display, egl_surface, egl_surface, context);
}

static struct gbm_bo *previous_bo = NULL;
static uint32_t previous_fb;

static void swap_buffers () {
	eglSwapBuffers (display, egl_surface);
	struct gbm_bo *bo = gbm_surface_lock_front_buffer (gbm_surface);
	uint32_t handle = gbm_bo_get_handle (bo).u32;
	uint32_t pitch = gbm_bo_get_stride (bo);
	uint32_t fb;
	drmModeAddFB (device, mode_info.hdisplay, mode_info.vdisplay, 24, 32, pitch, handle, &fb);
	drmModeSetCrtc (device, crtc->crtc_id, fb, 0, 0, &connector_id, 1, &mode_info);
	
	if (previous_bo) {
		drmModeRmFB (device, previous_fb);
		gbm_surface_release_buffer (gbm_surface, previous_bo);
	}
	previous_bo = bo;
	previous_fb = fb;
}

static void draw (float progress) {
	glClearColor (1.0f-progress, progress, 0.0, 1.0);
	glClear (GL_COLOR_BUFFER_BIT);
	swap_buffers ();
}

static void clean_up () {
	// set the previous crtc
	drmModeSetCrtc (device, crtc->crtc_id, crtc->buffer_id, crtc->x, crtc->y, &connector_id, 1, &crtc->mode);
	drmModeFreeCrtc (crtc);
	
	if (previous_bo) {
		drmModeRmFB (device, previous_fb);
		gbm_surface_release_buffer (gbm_surface, previous_bo);
	}
	
	eglDestroySurface (display, egl_surface);
	gbm_surface_destroy (gbm_surface);
	eglDestroyContext (display, context);
	eglTerminate (display);
	gbm_device_destroy (gbm_device);
}

int main () {
	device = open ("/dev/dri/card0", O_RDWR|O_CLOEXEC);
	find_display_configuration ();
	setup_opengl ();
	
	int i;
	for (i = 0; i < 600; i++)
		draw (i / 600.0f);
	
	clean_up ();
	close (device);
	return 0;
}
```
参考资料：
**[EGL Off-Screen rendering using GBM](https://blog.csdn.net/eydwyz/article/details/107046470)**
[GBM ext spec khronos](https://registry.khronos.org/EGL/extensions/MESA/EGL_MESA_platform_gbm.txt)
**[GBM for EGL (Linux)](https://winddoing.github.io/post/62365.html)**
[eglGetPlatformDisplay](https://registry.khronos.org/EGL/sdk/docs/man/html/eglGetPlatformDisplay.xhtml)
[OpenGL ES EGL eglGetDisplay](https://juejin.cn/post/7149533134185627679)
[wayland浅析之EGL、Opengles、GBM](https://blog.csdn.net/czg13548930186/article/details/131103472)
[代码-demo-例子-drm-gbm](https://github.com/eyelash/tutorials/blob/master/drm-gbm.c)
[代码-demo-例子-egl-gbm-render](https://raw.githubusercontent.com/Winddoing/CodeWheel/master/egl/egl_gbm_render.c)
[mesa-demos:eglkms.c](https://github.com/Distrotech/mesa-demos/blob/master/src/egl/opengl/eglkms.c)

#### xshell远程ssh连接服务器OpenGL初始化失败eglInitialize调用失败
暂时还不确定原因，此问题在安装了一个xmanager之后解决了。

#### 电脑运行时GLSL支持的版本太低
错误信息：error: GLSL 3.30 is not supported. Supported versions are: 1.10, 1.20, 1.30, 1.40, 1.00 ES, and 3.00 ES

解决方法：
1. 确保OpenGL驱动、环境等正常安装。[参考](https://crainyday.gitee.io/Ubuntu_004.html)
```
sudo apt-get install libgl1-mesa-dev
sudo apt-get install libglu1-mesa-dev
sudo apt-get install libegl1-mesa-dev
# 查看结果
glxinfo -B
```
2. 开启OpenGL直接渲染以及升级OpenGL。[参考](https://blog.csdn.net/GodNotAMen/article/details/125123186)
```shell
LIBGL_ALWAYS_INDIRECT=0
sudo add-apt-repository ppa:kisak/kisak-mesa
sudo apt update && sudo apt upgrade
```
3. 这个好像没用，但也记录一下
export MESA_GL_VERSION_OVERRIDE=3.3

#### 在linux server上使用OpenGL，无x-client
测试环境描述：后台服务器，只安装了x-server(xorg)。
问题：正常的eglGetDisplay(EGL_DEFAULT_DISPALY)后，eglInitialize()返回失败了。
解决方案：
1. 如果是通过xshell这种ssh方式连接服务器，则可以考虑安装xmanager，这个可以在远程（用户机器安装x-client），这样就可以初始化egl了。
2. 可以安装xvfb，然后启动Xvfb，这样也能正常初始化egl。
3. 使用eglGetPlatformDisplayEXT获取DISPLAY然后进行初始化。[参考](https://developer.nvidia.com/blog/egl-eye-opengl-visualization-without-x-server/)  [参考2](https://developer.nvidia.com/blog/linking-opengl-server-side-rendering/)

#### 未解决的疑惑
* 将Texture和Depth buffer绑定到Framebuffer时，可能导致出现glCheckFramebufferStatus检查时发现Attachment未成功，而将这个过程换个位置又可以成功，什么原因导致的还未搞清楚。
* 偶尔将Renderframe中的Depth buffer绑定到Framebuffer中时，glGetError会返回失败，但是glCheckFramebufferStatus将返回成功，而且绘制也是成功的。
* **虽然未证实，但是很大可能性是与InternalFormat的值有关**

#### 关于Internal format与Format
对于Texture来说，Opengl内部看到的类型是Internal format，这个值必须是合法的Internal format，否则可能导致**无法绑定到Framebuffer，或者无法往Texture拷贝内存更新内容**  
当将内存拷贝到Texture中或者从Texture中拷贝数据时，Format表示的是内存中的像素格式，如果这个设置错误，**可能导致颜色不正确**

#### 软件兼容性（opengl版本需求）
* 首先glCreateBuffers需要opengl 4.5及以上才支持，而glGenBuffers从2.0版本就已经开始支持
* 以下为各个opengl api对版本的要求，为了兼容性考虑尽量不要使用高版本的api
（**YY开播中最低版本需求是3.0**）
* 在一台win7的虚拟机中，检测得opengl版本为2.1，但是运行glGenFramebuffers确实成功的，这种情况很让人疑惑。glGenFramebuffers要求3.0。

API | 最低版本 | API | 最低版本
---|---|---|---
glCreateShader | 2.0 | glGenBuffers | 2.0
glCreateBuffers | 4.5 | glCreateProgram | 2.0
glGenTextures | 2.0 | glBindTexture | 2.0
glTexImage2D | 2.0 | glTexParameteri | 2.0
glDeleteTextures | 2.0 | glPixelStorei | 2.0 
glGenFramebuffers | 3.0 | glDeleteFramebuffers | 3.0
glShaderSource | 2.0 | glCompileShader | 2.0
glGetShaderiv | 2.0 | glGetShaderInfoLog | 2.0
glAttachShader | 2.0 | glLinkProgram | 2.0
glGetProgramiv | 2.0 | glGetProgramInfoLog | -
glDetachShader | 2.0 | glDeleteShader | 2.0
glClearColor | 2.0 | glClearDepth | 2.0
glClearDepthf | 4.1 | glClear | 2.0
glBindFramebuffer | 3.0 | glActiveTexture | 2.0
glCopyTexSubImage2D | 2.0 | glCopyTextureSubImage2D | 4.5
glDeleteVertexArrays | 3.0 | glDeleteProgram | 2.0
glDeleteBuffers | 2.0 | glDeleteRenderbuffers | 3.0
glGenVertexArrays | 3.0  | glBindVertexArray | 3.0
glBufferData | 2.0 | glFramebufferTexture2D | 2.0
glFramebufferRenderbuffer | 3.0 | glCheckFramebufferStatus | 3.0
glUseProgram | 2.0 | glVertexAttribPointer | 2.0
glEnableVertexAttribArray | 2.0 | glDisableVertexAttribArray | 2.0
glGetUniformLocation | 2.0  | glUniform1i | 2.0
glViewport | 2.0 | glDrawArrays | 2.0
glGetString | 2.0 | glGetIntegerv | 2.0
glReadPixels | 2.0 | glEnable | 2.0
glShadeModel | 2.1 | glDepthFunc | 2.0 

#### 推荐的编程习惯
* 调用bindxxx类似的接口之后，如果不在需要尽量解除绑定。这样可避免别的地方误用绑定的资源。
___
## glfw
### 帮助文档
* https://www.glfw.org/docs/latest/
* 头文件 glfw3.h

### 重要函数
> * glfwInit()
> * glfwWindowHint()
> * glfwCreateWindow()
> * glfwMakeContextCurrent()
> * glfwGetCurrentContext()
> * glfwPollEvents()
> * glfwSwapBuffers()
> * glfwTerminate()
### 其他
___
## glew (The OpenGL Extension Wrangler library)
### 帮助文档
* http://glew.sourceforge.net/
* 中文介绍 https://www.cnblogs.com/madfrog/archive/2010/06/25/1765259.html

## 其他
#### linux编译gl程序时提示 error: ‘glLinkProgram’ was not declared in this scope;
解决方法：
```
Nowadays, <GL/gl.h> typically only includes prototypes for the OpenGL 1.x API. Depending upon the platform, the library may only export the 1.1 functions; in which case, any other function has to be accessed via a pointer obtained via a platform-specific function (wglGetProcAddress on Windows, glXGetProcAddress on Unix/X11). Typically you use a loader such as GLEW, GL3W, GLAD etc to handle this part; the loader will provide a header containing prototypes for the additional function.

On Linux, libGL typically exports the entire OpenGL API (for some version), so you don’t need a loader. In that case, you just need

#define GL_GLEXT_PROTOTYPES
#include <GL/glext.h>
```

#### opengl跑分工具 glmark2。可直接安装 sudo apt install glmark2
#### EGL初始化(eglInitialize)失败
表现：
1. 使用EGL_DEFAULT_DISPLAY初始化EGL，调用eglInitialize会返回失败
2. 控制台打印输出：libEGL warning: egl: failed to create dri2 screen
3. 此时使用glxinfo查看信息也是失败的，比如运行命令：glxinfo -B。控制台打印：Error: unable to open display :0.0
    > 这个是因为使用ssh连接，并且ssh配置文件中把forwarding关闭了，可以修改ssh配置文件/etc/ssh/sshd_config，设置 X11Forwarding yes。[参考](https://linuxconfig.org/fixing-the-cannot-open-display-error-on-linux)
4. 使用sudo eglinfo收集信息，GBM platform模式下有错误提示：kmsro: driver missing
尝试：
+ 安装server版本的NVIDIA驱动。nvidia-driver-470-server 

==尝试了很多的方法，但是此问题其实并没有解决。以EGL_DEFAULT_DISPLAY调用eglInitialize在没有显示器的服务器上还是失败了。后来使用了eglGetPlatformDisplay获取display对象才解决问题。==

#### 从PBO拷贝数据到Texture时失败
+ 表现：使用PBO，数据格式为GL_RED, 然后调用glTexImage2D或者glTexSubImage2D将PBO中的数据拷贝到Texture时失败了，glGetError()返回GL_INVALID_OPERATION错误码。
+ [参考](https://stackoverflow.com/questions/27294882/glteximage2d-fails-with-error-1282-using-pbo-bad-texture-resolution)
+ 原因：PBO的每行的长度并没有4字节对齐，因为Texture从PBO拷贝数据时需要4个字节对齐，而PBO因为格式是GL_RED，所以容易长度未对齐，此时拷贝数据将导致失败。
+ glTexSubImage2D的错误码说明
```
Errors
GL_INVALID_ENUM is generated if target or the effective target of texture is not GL_TEXTURE_2D, GL_TEXTURE_CUBE_MAP_POSITIVE_X, GL_TEXTURE_CUBE_MAP_NEGATIVE_X, GL_TEXTURE_CUBE_MAP_POSITIVE_Y, GL_TEXTURE_CUBE_MAP_NEGATIVE_Y, GL_TEXTURE_CUBE_MAP_POSITIVE_Z, GL_TEXTURE_CUBE_MAP_NEGATIVE_Z, or GL_TEXTURE_1D_ARRAY.
GL_INVALID_OPERATION is generated by glTextureSubImage2D if texture is not the name of an existing texture object.
GL_INVALID_ENUM is generated if format is not an accepted format constant.
GL_INVALID_ENUM is generated if type is not a type constant.
GL_INVALID_VALUE is generated if level is less than 0.
GL_INVALID_VALUE may be generated if level is greater than log2 max, where max is the returned value of GL_MAX_TEXTURE_SIZE.
GL_INVALID_VALUE is generated if xoffset<−b, (xoffset+width)>(w−b), yoffset<−b, or (yoffset+height)>(h−b), where w is the GL_TEXTURE_WIDTH, h is the GL_TEXTURE_HEIGHT, and b is the border width of the texture image being modified. Note that w and h include twice the border width.
GL_INVALID_VALUE is generated if width or height is less than 0.
GL_INVALID_OPERATION is generated if the texture array has not been defined by a previous glTexImage2D operation.
GL_INVALID_OPERATION is generated if type is one of GL_UNSIGNED_BYTE_3_3_2, GL_UNSIGNED_BYTE_2_3_3_REV, GL_UNSIGNED_SHORT_5_6_5, or GL_UNSIGNED_SHORT_5_6_5_REV and format is not GL_RGB.
GL_INVALID_OPERATION is generated if type is one of GL_UNSIGNED_SHORT_4_4_4_4, GL_UNSIGNED_SHORT_4_4_4_4_REV, GL_UNSIGNED_SHORT_5_5_5_1, GL_UNSIGNED_SHORT_1_5_5_5_REV, GL_UNSIGNED_INT_8_8_8_8, GL_UNSIGNED_INT_8_8_8_8_REV, GL_UNSIGNED_INT_10_10_10_2, or GL_UNSIGNED_INT_2_10_10_10_REV and format is neither GL_RGBA nor GL_BGRA.
GL_INVALID_OPERATION is generated if format is GL_STENCIL_INDEX and the base internal format is not GL_STENCIL_INDEX.
GL_INVALID_OPERATION is generated if a non-zero buffer object name is bound to the GL_PIXEL_UNPACK_BUFFER target and the buffer object's data store is currently mapped.
GL_INVALID_OPERATION is generated if a non-zero buffer object name is bound to the GL_PIXEL_UNPACK_BUFFER target and the data would be unpacked from the buffer object such that the memory reads required would exceed the data store size.
GL_INVALID_OPERATION is generated if a non-zero buffer object name is bound to the GL_PIXEL_UNPACK_BUFFER target and pixels is not evenly divisible into the number of bytes needed to store in memory a datum indicated by type.
```

#### OpenGL使用的是CPU渲染没用到GPU渲染
[libglvnd](https://gitlab.freedesktop.org/glvnd/libglvnd)
表现：
1. 在使用EGL_DEFAULT_DISPLAY初始化EGL时失败了，调用eglInitialize()返回失败。
2. 使用eglQueryDevicesEXT枚举系统中EGL device总数，然后再循环用 eglGetPlatformDisplay()获取EGL_PLATFORM_DEVICE_EXT类型的display用于初始化EGL。
3. 通过eglGetPlatformDisplay()轮询的display可以成功初始化EGL。但是此时使用的CPU进行渲染，而不是GPU。
4. 环境方面：/usr/share/glvnd/egl_vendor.d/ 下只有一个json文件，即“50_mesa.json”。NVIDIA显卡，且显卡驱动是正常安装的。NVIDIA的EGL库是存在的，文件为：/usr/lib/x86_64-linux-gnu/libEGL_nvidia.so.520.61.05。

原因：可能是因为linux系统是直接拷贝的镜像，导致NVIDIA并没有正确注册到[EGL的IDC列表](https://gitlab.freedesktop.org/glvnd/libglvnd/-/blob/master/src/EGL/icd_enumeration.md)中。导致通过eglGetPlatformDisplay()时没有找到与NVIDIA显卡对应的Platform Display对象。此时创建的就是与mesa egl库对应的Display对象，用此Display对象调用eglInitialize()可以成功，但是后续的OpenGL请求将会调用Mesa的OpenGL实现，而不是NVIDIA的驱动的实现。这样就只用到了CPU渲染。

解决方式：
在/usr/share/glvnd/egl_vendor.d/下创建一个NVIDIA vendor对应的json文件，命名为：40_nvidia.json。文件内容如下：
```
{
    "file_format_version" : "1.0.0",
    "ICD" : {
        "library_path" : "libEGL_nvidia.so.0"
    }
}
```
此json文件的具体规则可以[参考](https://gitlab.freedesktop.org/glvnd/libglvnd/-/blob/master/src/EGL/icd_enumeration.md#icd-installation)

## 参考资料
+ [learnopengl-cn](https://learnopengl-cn.readthedocs.io/zh/latest/) -- 很好的入门教程
+ [EGLStream](https://docs.nvidia.com/drive/drive_os_5.1.6.1L/nvvib_docs/index.html#page/DRIVE_OS_Linux_SDK_Development_Guide/Windows%20Systems/window_system_egl.html) ，很有意思的东西。连接渲染端作为内容生产者以及一个消费者。