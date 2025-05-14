## 编程
包含以下头文件：#include <immintrin.h>
编译选项：
-mavx512f 支持AVX 512
支持 AVX-512VBMI ： -mavx512vbmi -mavx512vl
-mfma  支持fma指令
支持avx2,256bit: -mavx2 (gcc/clang)、 /arch:AVX2 (msvc)
在cmake的CMakeLists.txt中添加： add_compile_options(-mfma)

## SSE3
| 指令 | 格式 | 运算 |
| -- | -- | -- |
| PCMPEQB <br> PCMPEQW <br> PCMPEQD | PCMPEQB src dst | 逐个字节比较src与dst的值，dst[i] = src[i]==dst[i] ? 0xff : 0 |

## AVX2
### 数值转换
+ _mm256_cvtepu8_epi32
将 8 位无符号整数（uint8_t）零扩展（Zero Extension）为 32 位有符号整数（int32_t）。
输入：一个 128 位寄存器 a（__m128i），包含 16 个 8 位无符号整数（uint8_t）。
输出：一个 256 位寄存器（__m256i），包含 8 个 32 位有符号整数（int32_t），由输入的低 8 个字节（a[0] 到 a[7]）零扩展得到。高 8 个字节（a[8] 到 a[15]）被忽略。

### 移位
+ _mm_srli_si128
将__m128i逻辑右移n个字节


| 指令 | 格式 | 运算 |
| -- | -- | -- |

## 参考
[intel指令官方文档](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#expand=3828,301,2553&cats=Load&ig_expand=3981,4031)
[SIMD指令集 - MMX技术(4) - 比较指令](https://blog.csdn.net/qq_43401808/article/details/86663896)
[x86平台SIMD编程入门](https://www.cnblogs.com/moonzzz/p/17806554.html)