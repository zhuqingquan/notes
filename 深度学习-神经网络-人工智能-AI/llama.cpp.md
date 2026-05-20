# llama.cpp备忘

## 命令行
```shell
CUDA_VISIBLE_DEVICES=1 ./llama-cli --model /mnt/ext_file/models/Qwen3.6-35B-A3B/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf --mmproj /mnt/ext_file/models/Qwen3.6-35B-A3B/mmproj-F16.gguf
```
vscode launch.json
```json
{
        "name": "llama-cli gemme",
        "type": "cppdbg",
        "request": "launch",
        "program": "${workspaceFolder}/build-cuda/bin/llama-cli",
        "args": [
            "--model",
            "/mnt/ext_file/models/gemma-4/gemma-4-E2B-it-UD-Q4_K_XL.gguf",
            "--mmproj",
            "/mnt/ext_file/models/gemma-4/mmproj-F16.gguf",
        ],
        "stopAtEntry": false,
        "cwd": "${workspaceFolder}/build-cuda/bin/",
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
        "setupCommands": [
            {
                "description": "为 gdb 启用整齐打印",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description": "将反汇编风格设置为 Intel",
                "text": "-gdb-set disassembly-flavor intel",
                "ignoreFailures": true
            }
        ]
    }
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
        new llama_model(param)
```

#### llama-model.cpp -- class llama_model
主要方法：
llama_model::load_hparams(llama_model_loader& ml)
llama_model::load_vocab(llama_model_loader & ml)
llama_model::load_tensors(llama_model_loader & ml)
llama_model::build_graph(const llm_graph_params & params)

#### src/models
每一种类型的模型都会对应此文件夹内一种builder，使用此builder根据模型的内部结构和参数设置进行模型的graph创建。
比如gemma-4使用的是gemme4_iswa。对应源码：`gemma4-iswa.cpp`。

```
llm_graph_context
llm_build_gemma4_iswa
```

#### struct llama_context -- src/llama-context.h
#### src/llama-graph.h
struct llama_graph_context
struct llm_graph_result

### ggml
#### ggml_cgraph -- ggml/src/ggml-impl.h
    根据代码分析，ggml_cgraph 的设计和内存布局如下：
    
    数据结构定义
    
    在 ggml/src/ggml-impl.h 第 329 行：
    
    ```c
    struct ggml_cgraph {
        int size;                          // 最大节点数
        int n_nodes;                      // 当前使用的节点数
        int n_leafs;                      // 当前使用的叶子数
    
        struct ggml_tensor ** nodes;       // 可变张量（计算图输出）
        struct ggml_tensor ** grads;      // 梯度张量（grads=true时）
        struct ggml_tensor ** grad_accs;  // 梯度累加器（grads=true时）
        struct ggml_tensor ** leafs;     // 常量张量（参数）
        int32_t             * use_counts; // 张量使用计数
    
        struct ggml_hash_set visited_hash_set;  // 哈希集合
    
        enum ggml_cgraph_eval_order order;
        uint64_t uid;
    };
    ```
    
    内存布局
    
    ggml_new_graph_custom 将整个 cgraph 分配为 一块连续内存，布局如下：
    
    
    | ggml_cgraph 结构体本身 |
    | [对齐填充]              |
    | nodes[0..size-1]        |  size  sizeof(tensor)
    | leafs[0..size-1]        |  size  sizeof(tensor)
    | use_counts[0..hash_size-1]  |  hash_size * sizeof(int32_t)
    | hash_keys[0..hash_size-1]   |  hash_size  sizeof(tensor)
    | grads[0..hash_size-1]   |  hash_size  sizeof(tensor)  (grads=true)
    | grad_accs[0..hash_size-1] |  hash_size  sizeof(tensor) (grads=true)
    | hash_used (bitset)      |  ggml_bitset_size(hash_size)
    
    
    - hash_size = ggml_hash_size(size * 2) — 哈希表大小是节点容量的 2 倍
    
    设计原因
    
    1. 一次性分配，简化生命周期管理
    
    整个 cgraph 包括所有元数据数组都分配在 ggml_context 的内存缓冲区中，只需一次 ggml_new_object() 调用，释放也只需一次。避免了多次 malloc 开销。
    
    2. 哈希表实现 O(1) 张量查找
    
    前向/反向计算时需要快速判断某个张量是否已被访问过。哈希表通过张量指针地址（p >> 4）作为 key，实现常数时间查找。visited_hash_set 用于图遍历时去重。
    
    3. nodes vs leafs 分离
    
    - leafs：常量参数（如权重），图计算时不会变
    - nodes：计算产生的中间结果，张量数据会在图求值时更新
    
    分离存储使得后端（如 CUDA、Metal）可以分别处理只读数据和可写数据。
    
    4. 哈希表容量 2 倍设计
    
    负载因子保持在 ~50% 以下，减少哈希冲突，提高查找效率。这对大图（如 LLM 数千层）的遍历性能至关重要。
    
    5. grads/grad_accs 可选
    
    grads=true 时才分配梯度相关数组。用户只需要前向计算时（如推理），可以跳过这部分内存分配。
    
    6. use_counts 配合哈希表
    
    每个哈希槽存储对应张量的使用计数，用于引用计数和内存管理。

#### llama_kv_cache_iswa_context 分析
    
    作用
    
    llama_kv_cache_iswa_context 是 ISWA (Indexable Sliding Window Attention) KV Cache 的上下文对象，用于管理包含 SWA（Sliding Window Attention）层的模型的 KV 缓存。它封装了对两类底层 kv_cache 的并发管理：
    - 非 SWA 层（普通 Attention）→ kv_base
    - SWA 层（滑动窗口 Attention）→ kv_swa
    
    用法
    
    用户不直接构造，而是通过 llama_kv_cache_iswa 的工厂方法创建：
    
    cpp
    // 三种初始化模式
    llama_memory_context_ptr init_batch(...);   // 批量推理
    llama_memory_context_ptr init_full();        // 完整缓存
    llama_memory_context_ptr init_update(...);    // 更新模式
    
    // 遍历
    while (ctx->next()) {         // 前进到下一个 ubatch
        ctx->apply();              // 应用当前 batch 的 KV 写入
    }
    
    
    使用示例（批量推理）：
    cpp
    auto ctx = kv->init_batch(balloc, n_ubatch, embd_all);
    while (ctx->next()) {
        const auto & ubatch = ctx->get_ubatch();
        const auto * base = ctx->get_base();  // 非SWA层上下文
        const auto * swa  = ctx->get_swa();   // SWA层上下文
        ctx->apply();
    }
    
    
    算法原理
    
    核心问题：SWA 层只保留最近 W 个 token 的 KV，而非 SWA 层需要保留完整上下文。两类层的缓存容量和使用模式不同，需要分别管理。
    
    解决方案：
    1. 双缓存分离 — 普通层用完整 kv_size，SWA 层用缩小版 kv_size_swa（对齐到 256）
    2. 分批处理 — 将大批次分割成多个小 ubatch，分别用 ctx_base 和 ctx_swa 处理
    3. 统一接口 — 对上屏蔽两个缓存的细节，提供统一的 next() / apply() 接口
    
    实现方式
    
    1. 构造时分离（llama_kv_cache_iswa 构造）
    
    cpp
    // 第 30-36 行：链式过滤器，分离 SWA 层
    const layer_filter_cb filter_base = & {
        return !model.hparams.is_swa(il);  // 非SWA
    };
    const layer_filter_cb filter_swa  = & {
        return model.hparams.is_swa(il);    // SWA
    };
    
    // 第 46-58 行：SWA 缓存大小计算
    uint32_t size_swa = GGML_PAD(std::min(size_base, hparams.n_swa*(unified ? n_seq_max : 1) + n_ubatch), 256);
    
    
    2. 批处理分割策略（init_batch）
    
    尝试两种分割策略（按优先级）：
    
    策略一：简单分割 (split_simple) — 适用于 unified=true，每个 token 可自由映射到任何位置
    
    策略二：等分分割 (split_equal) — 将缓存等分成连续区块，!unified 时强制等分保证不同序列不重叠
    
    cpp
    // 第 166-167 行：成功则创建上下文
    return std::make_unique<llama_kv_cache_iswa_context>(
        this, std::move(sinfos_base), std::move(sinfos_swa), std::move(ubatches));
    
    
    3. 遍历逻辑（next / apply）
    
    cpp
    // llama_kv-cache-iswa.cpp 第 286-297 行
    bool llama_kv_cache_iswa_context::next() {
        ctx_base->next();   // 底层base缓存前进
        ctx_swa ->next();   // 底层swa缓存前进
        if (++i_next >= ubatches.size()) {
            return false;   // 所有batch处理完毕
        }
        return true;
    }
    
    bool llama_kv_cache_iswa_context::apply() {
        bool res = true;
        res = res & ctx_base->apply();  // 应用base写入
        res = res & ctx_swa ->apply();  // 应用swa写入
        return res;
    }
    
    
    数据结构
    
    cpp
    class llama_kv_cache_iswa_context : public llama_memory_context_i {
        size_t i_next;                              // 当前处理的ubatch索引
    
        std::vector<llama_ubatch> ubatches;        // 所有待处理的子批次
    
        const llama_memory_context_ptr ctx_base;    // 非SWA层缓存上下文
        const llama_memory_context_ptr ctx_swa;     // SWA层缓存上下文
    
        const llama_memory_status status;           // 整体状态
    };
    
    
    各字段含义
    
    | 字段 | 含义 |
    |------|------|
    | i_next | 下一个要处理的 ubatch 索引 |
    | ubatches | 从原始大批次分割出来的子批次数组 |
    | ctx_base | 持有对 kv_base 的 slot 映射信息（哪些 cell 给哪些 token） |
    | ctx_swa | 持有对 kv_swa 的 slot 映射信息 |
    | status | 组合状态：SUCCESS / PARTIAL / FAILED |
    
    设计原因
    
    1. SWA 与普通 Attention 解耦 — 两类层的缓存大小、生命周期不同，强制共享会浪费内存或限制序列长度
    2. 256 对齐优化 — SWA 缓存对齐到 256 是硬件友好的（SIMD 友好、页对齐）
    3. 统一遍历 — next()/apply() 接口让上层（llama_graph）只需一个循环即可驱动两类缓存
    4. 状态组合 — ctx_base 和 ctx_swa 任一失败则整体失败，避免不一致
