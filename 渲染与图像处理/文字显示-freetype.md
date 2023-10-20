### 使用OpenGL显示freetype生成的文字
[教程](https://learnopengl-cn.readthedocs.io/zh/latest/06%20In%20Practice/02%20Text%20Rendering/)

#### 字体glyph相关属性
| 属性 | 获取方式 | 生成位图描述 |
| --- | --- | --- |
width | face->glyph->bitmap.width | 宽度，单位:像素 
height | face->glyph->bitmap.rows | 高度，单位:像素 
bearingX| face->glyph->bitmap_left| 水平位置(相对于起点origin)，单位:像素 
bearingY| face->glyph->bitmap_top | 垂直位置(相对于基线Baseline)，单位:像素 
advance | face->glyph->advance.x | 起点到下一个字形的起点间的距离(单位:1/64像素)

### 需要解决的问题
#### 中文的显示
#### 如何显示一句话
#### 更漂亮的字
实现方式：
1. 使用字体文件
    + [中文开源字体集 Open Source Fonts Collection for Chinese](https://drxie.github.io/OSFCC/)
2. 

#### 阴影、描边（text halos）、3D字体
+ 参考资料[AlphaTestedMagnification](#参考资料)中有关于描边和阴影的一种实现。
+ 这里包含很多字体特效的shader，(http://digitalnativestudios.com/textmeshpro/docs/shaders/)

#### 文字的缩放与旋转
在对文字进行缩放或者旋转显示时会变得模糊或者产生锯齿，可用使用有向距离场(Signed Distance Fields)技术进行解决。
[Font Rendering is Getting Interesting 汇总介绍了很多先进的文件渲染技术](https://aras-p.info/blog/2017/02/15/Font-Rendering-is-Getting-Interesting/)
[Drawing Text with Signed Distance Fields in Mapbox GL](https://blog.mapbox.com/drawing-text-with-signed-distance-fields-in-mapbox-gl-b0933af6f817)
##### Signed Distance Fields的shader例子
```glsl
precision mediump float;

uniform sampler2D u_texture;
uniform vec4 u_color;
uniform float u_buffer;
uniform float u_gamma;

varying vec2 v_texcoord;

void main() {
float dist = texture2D(u_texture, v_texcoord).r;
float alpha = smoothstep(u_buffer — u_gamma, u_buffer + u_gamma, dist);
gl_FragColor = vec4(u_color.rgb, alpha * u_color.a);
}
```
[libGDX库中对Signed Distance Fields的实现](https://libgdx.com/wiki/graphics/2d/fonts/distance-field-fonts)

##### multi-channel distance field比SDF更进一步，效果更好
可用于解决SDF中尖角（sharp corners）被磨平的问题。
“multi-channel distance field” by Viktor Chlumský which seems to be pretty nice
**[msdfgen库](https://github.com/Chlumsky/msdfgen)**

### 其他
+ FT_Load_Glyph接口调用之后，就可以获取到文字的分辨率大小。即使此时还没有生成文字对应的bitmap。
```
    // 将字符存储到字符表中备用
    Character character = {
        texture, 
        glm::ivec2(face->glyph->bitmap.width, face->glyph->bitmap.rows),
        glm::ivec2(face->glyph->bitmap_left, face->glyph->bitmap_top),
        face->glyph->advance.x
    };
```

#### 生成文字的PNG再渲染
+ [makeglfont](https://github.com/raphm/makeglfont)
+ [Freetype GL](https://github.com/rougier/freetype-gl)   A small library for displaying Unicode in OpenGL using a single texture and a single vertex buffer.

### 参考资料
+ [freetype显示中文--韦东山](https://blog.csdn.net/qq_22655017/article/details/90034431)
+ [AlphaTestedMagnification.pdf](https://github.com/Michaelangel007/game_dev_pdfs/blob/master/graphics/signed_distance_field/SIGGRAPH2007_AlphaTestedMagnification.pdf)