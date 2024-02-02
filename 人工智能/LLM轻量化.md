# LLM轻量化

## 模型对比

| 模型          | chatglm3                                         | qwen                                                                             |
| ------------- | ------------------------------------------------ | -------------------------------------------------------------------------------- |
| 量化          | 4bit                                             | 4bit                                                                             |
| 量化实现      | [chatglm](https://github.com/li-plus/chatglmcpp) | [llama-cpp-python](https://github.com/abetlen/llama-cpp-python#function-calling) |
| function call | ✅                                                | ✅                                                                                |
| file size     | 3.3G                                             | 4.6G                                                                             |
| mem usage     | 4G                                               |                                                                                  |
| cpu           | 1600% （默认占用一半的cpu）                      |                                                                                  |