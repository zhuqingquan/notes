# llama.cpp备忘

## 命令行
```shell
CUDA_VISIBLE_DEVICES=1 ./llama-cli --model /mnt/ext_file/models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf --mmproj /mnt/ext_file/models/Qwen3.6-35B-A3B/mmproj-F16.gguf
```

## 代码分析
### llama.cli加载与环境
模型加载
```
server_context::load_model(common_params & params)
server_context_impl(common_params & params)
common_init_from_params(common_params & params) // common/common.cpp
common_init_result::common_init_result(common_params & params)
    auto mparams = common_model_params_to_llama(params);
    auto cparams = common_context_params_to_llama(params);
    if (params.fit_params) llama_params_fit()
    llama_model * model = llama_model_load_from_file(params.model.path.c_str(), mparams);
    llama_model * llama_model_load_from_file_impl()  // src/llama.cpp
```