## 代码构建
主要参考`docs/build.md`
```shell
# build with CUDA
cmake -B build-cuda -DCMAKE_BUILD_TYPE=Debug -DGGML_CUDA=ON
cmake --build build --config Release
```