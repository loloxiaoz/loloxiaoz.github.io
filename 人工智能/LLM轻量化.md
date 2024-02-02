# LLM轻量化

# chatbot 技术方案

## 功能

## 技术栈

开发框架: langchain4j
模型微调: LLaMa-Factory
模型量化: llama.cpp

### 模型对比

- 模型量化: 大模型量化是指通过减少神经网络中权重和激活值的位数，从而减小模型的存储空间和计算开销。该过程通常涉及将浮点数参数转换为较低位数的整数或定点数，以减少模型的内存占用和计算复杂度。
  - 权重量化: 将模型的权重从浮点数表示转换为较低位数的整数或定点数
  - 激活值量化: 对神经网络中的激活值进行类似处理
  - 量化训练: 在训练模型时，使用量化后的权重和激活值进行前向传播和反向传播，有助于模型适应于低位数表示
- 模型微调: 微调少量或额外的模型参数，固定大部分预训练模型（LLM）参数，从而大大降低了计算和存储成本，同时，也能实现与全量参数微调相当的性能

| 模型          | chatglm3                                         | qwen                                                                             |
| ------------- | ------------------------------------------------ | -------------------------------------------------------------------------------- |
| 量化          | 4bit                                             | 4bit                                                                             |
| 量化实现      | [chatglm](https://github.com/li-plus/chatglmcpp) | [llama-cpp-python](https://github.com/abetlen/llama-cpp-python#function-calling) |
| function call | ✅                                                | ✅                                                                                |
| file size     | 3.3G                                             | 4.6G                                                                             |
| mem usage     | 4G                                               | 5.6g                                                                             |
| cpu           | 1600% （默认占用一半的cpu）                      | 1600%(默认占用一半的cpu)                                                         |

### 模型训练

模型微调(LLaMa-Factory) -> 模型合并(LLaMa-Factory) -> 模型评估(LLaMa-Factory) -> 模型量化(llama.cpp) -> 模型部署(llama-cpp-python)

1. 模型微调  

使用LLaMa-Factory项目进行lora微调, Vram资源占用超过了32GB, 使用deepspeed + zero3进行加速和降低显存要求

训练数据源包括

- 自我认知(小禹助手)
- 工具使用

2. 模型合并

使用LLaMa-Factory提供的export.py脚本进行导出, 将原始7b模型(15G) 和 finetune（19M）模型合并成hf格式的模型

```shell
CUDA_VISIBLE_DEVICES=0 python src/evaluate.py \
--model_name_or_path $ft_model_merge_path \
--template qwen \
--finetuning_type lora \
--task ceval \
--split validation \
--task_dir ./evaluation/ \
--lang zh \
--n_shot 5 \
--batch_size 4
```

3. 模型评估

```shell
CUDA_VISIBLE_DEVICES=0 python src/evaluate.py \
--model_name_or_path $ft_model_merge_path \
--template qwen \
--finetuning_type lora \
--task ceval \
--split validation \
--task_dir ./evaluation/ \
--lang zh \
--n_shot 5 \
--batch_size 4
```

4. 模型量化

- 使用llama.cpp提供的convert工具，先将hf转换成gguf格式
- 使用llama.cpp提供的quantize工具，进行q4_0级别的量化

```shell
#模型转换
python3 convert-hf-to-gguf.py  ../LLaMA-Factory/output/Qwen-7B-Chat-sft-merge/ --outfile qwen/qwen-7b-chat-ft.gguf
#模型量化
./quantize qwen/qwen-7b-chat-ft.gguf qwen/qwen-7b-chat-ft-Q4_0.gguf q4_0
```

5. 模型部署

使用llama-cpp-python提供的脚本, 启动openapi兼容的接口格式

```shell
#cli 模式
./main -m qwen/qwen-7b-chat-ft-Q4_0.gguf -ins -c 2048
#server 模式
python3 -m llama_cpp.server --model Qwen-7B-Chat.Q4_K_M.gguf  --host xxxx --chat_format qwen
```

## DFX

### 扩展性

1、多模型兼容
2、增加本地知识库的支持

### 性能

- 常见问题缓存、问题预计算
- 并发限制

### 安全性

敏感问题过滤

- AC自动机
- DFA过滤算法 

## 数据集

- attck信息集