## DeepSeek

DeepSeek：专注于 大模型训练与优化 的工具，主要帮助用户高效训练和部署定制化的大型语言模型（LLM）

DeepSeek-R1 系列

## Ollama

选用Ollama 作为 DeepSeek 大模型运行框架

找到 DeepSeek R1：<https://ollama.com/library/deepseek-r1>

Ollama 的官网直接打开 DeepSeek 大模型的拉取链接，在 models 里选择 deepseek-r1 即可 <https://ollama.com/library/deepseek-r1:70b>

网络不好导致拖不动 DeepSeek 模型、或者单纯想在一台隔离网络的单机上部署 DeepSeek，可以用迅雷下载所需的大模型，并用手动导入到 ollama 指定的文件夹内

先去  搜索所需的 DeepSeek 模型，搜索的时候，加上关键词 GGUF（ollama 支持的模型文件格式）

- 国内大模型源文件下载地址 <https://modelscope.cn/models>
- 国外大模型源文件下载地址 <https://huggingface.co/models>

Ollama 是一个开源的AI工具，支持本地运行各种模型，包括 GPT-4、DeepSeek 等。要用 DeepSeek 就必须要安装运行框架

部署 helm charts：<https://github.com/otwld/ollama-helm>

推送到本地仓库

```bash
```

## Open-WebUI

Open-WebUI：是一个开源的大模型交互界面，为用户提供友好的 Web 界面，方便与大模型进行交互