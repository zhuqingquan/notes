### cv::Mat
支持的数据格式：CV_{8U,16S,16U,32S,32F,64F}C{1,2,3}
```
// 初始化
cv::Mat srcMat(h, w, CV_8UC3, (uint8_t*)buffer.c_str());        // 使用已有数据初始化，不复制数据。注意数据释放问题
cv::Mat srcMat(h, w, CV_8UC3);                                  // 初始化，没有数据
cv::Mat m1 = cv::Mat_<double>(2, 3, CV_64FC1) << pmat[0], pmat[1], pmat[2], pmat[3], pmat[4], pmat[5]); // 创建对象时直接初始化
```
#### 经验
+ Mat赋值语句并没有复制内容，2个Mat指向相同的数据，此时注意修改一个对象的值另一个对象的值其实也发生了改变
```
cv::Mat m1 = cv::Mat_<double>(2, 3, CV_64FC1) << pmat[0], pmat[1], pmat[2], pmat[3], pmat[4], pmat[5]);
cv::Mat m2 = m1;  // 注意此时m2中的内容是通过指针指向m1的buffer
cv::Mat m3 = m1.clone();        // 调用clone()才会真正复制m1数据
```

### opencv imgproc
#### cv::warpAffine
[解析](https://blog.csdn.net/qq_22734027/article/details/134482760#:~:text=warpAffine%20%28%29%20%E6%98%AF%20OpenCV%20%E5%BA%93%E4%B8%AD%E7%9A%84%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%EF%BC%8C%E7%94%A8%E4%BA%8E%E8%BF%9B%E8%A1%8C%E4%BA%8C%E7%BB%B4%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2%E3%80%82%20%E8%AF%A5%E5%87%BD%E6%95%B0%E5%B0%86%E8%BE%93%E5%85%A5%E5%9B%BE%E5%83%8F%E6%98%A0%E5%B0%84%E5%88%B0%E8%BE%93%E5%87%BA%E5%9B%BE%E5%83%8F%EF%BC%8C%E5%BA%94%E7%94%A8%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2%E3%80%82,%E5%87%BD%E6%95%B0%E5%8E%9F%E5%9E%8B%E5%A6%82%E4%B8%8B%EF%BC%9A%20src%EF%BC%9A%E8%BE%93%E5%85%A5%E5%9B%BE%E5%83%8F%EF%BC%8C%E5%BF%85%E9%A1%BB%E6%98%AF%E5%8D%95%E9%80%9A%E9%81%93%E6%88%96%E4%B8%89%E9%80%9A%E9%81%93%E7%9A%848%E4%BD%8D%E6%88%9632%E4%BD%8D%E6%B5%AE%E7%82%B9%E5%9E%8B%E5%9B%BE%E5%83%8F%E3%80%82%20dst%EF%BC%9A%E8%BE%93%E5%87%BA%E5%9B%BE%E5%83%8F%EF%BC%8C%E5%85%B6%E5%A4%A7%E5%B0%8F%E5%92%8C%E7%B1%BB%E5%9E%8B%E4%B8%8E%E8%BE%93%E5%85%A5%E5%9B%BE%E5%83%8F%E7%9B%B8%E5%90%8C%E3%80%82%20mat%EF%BC%9A2x3%E7%9A%84%E5%8F%98%E6%8D%A2%E7%9F%A9%E9%98%B5%E3%80%82%20dsize%EF%BC%9A%E8%BE%93%E5%87%BA%E5%9B%BE%E5%83%8F%E7%9A%84%E5%A4%A7%E5%B0%8F%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%E4%B8%BA%20Size%28%29%20%EF%BC%8C%E5%88%99%E8%BE%93%E5%87%BA%E5%9B%BE%E5%83%8F%E7%9A%84%E5%A4%A7%E5%B0%8F%E5%B0%86%E4%B8%8E%E8%BE%93%E5%85%A5%E5%9B%BE%E5%83%8F%E7%9B%B8%E5%90%8C%E3%80%82)

### 编译错误
+ undefine symbole; ffi_type_pointer...
可能是因为安装了anaconda导致的。anaconda/lib中的libffi.so.7实际上链接到了8版本。
https://blog.csdn.net/qq_38606680/article/details/129118491


### 安装备忘
```
# Download and unpack sources 或者 git clone
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip
unzip opencv.zip
 
# Create build directory
mkdir -p build && cd build
 
# Configure
cmake  ../opencv-4.x -DCMAKE_INSTALL_PREFIX=./output
 
# Build
cmake --build .
```

### 函数
| 函数名 | 功能 | 参考 | 示例 |
| ---- | ---- | ---- | ---- |
| arcLength | 计算 轮廓的周长 | | |
| approxPolyDP | 生成逼近曲线 | [1](https://www.cnblogs.com/bjxqmy/p/12347265.html) | |
| watershed | 基于分水岭算法的图像分割 | [api](https://docs.opencv.org/4.x/d3/d47/group__imgproc__segmentation.html#ga3267243e4d3f95165d55a618c65ac6e1) <br> [例子](https://docs.opencv.org/4.x/d2/dbd/tutorial_distance_transform.html) <br> [基于分水岭算法的图像分割](https://www.bookstack.cn/read/opencv-doc-zh-4.0/docs-4.0.0-4.15-tutorial_py_watershed_segmentation.md) | |
|  findContours | | [api说明](https://docs.opencv.org/3.4/d4/d73/tutorial_py_contours_begin.html) | |
|  drawContours | 根据轮廓数据在图像上绘制轮廓线条或填充轮廓。 | [api说明](https://docs.opencv.org/3.4/d4/d73/tutorial_py_contours_begin.html) | |
| threshold | 二值化函数 | [API说明](https://docs.opencv.org/3.4/d7/d4d/tutorial_py_thresholding.html) | |
| 几何绘制 | 绘制直线、矩形、圆等 | [api](https://docs.opencv.org/4.x/dc/da5/tutorial_py_drawing_functions.html) | |
### 类
| 类名 | 功能 | 方法 | 示例 |
| ---- | ---- | ---- | ---- |
| [QRCodeDetector](https://docs.opencv.org/3.4/de/dc3/classcv_1_1QRCodeDetector.html#a64373f7d877d27473f64fe04bb57d22b) | 检测二维码内容和位置信息 | decode() <br> detectAndDecode() <br> detect() | data, vertices, _ = detector_cv.detectAndDecode(image) |