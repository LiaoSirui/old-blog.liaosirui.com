## GGUF

GGUF (GPT-Generated Unified Format) 是一种文件格式，用于保存经过微调的语言模型。这种格式旨在帮助用户方便地在不同的平台和环境之间共享和导入模型。它支持多种量化格式，可以有效减少模型文件的大小。

它的前身是 GGML(GPT-Generated Model Language)，是专门为了机器学习而设计的 Tensor 库，目的是为了有一个单文件的格式，并且易在不同架构的 CPU 以及 GPU 上可以推理，但后续由于开发遇到了灵活性不足、相容性及难以维护的问题。

网络不好导致拖不动模型、或者单纯想在一台隔离网络的单机上部署模型，可以下载所需的大模型，并用手动导入到 ollama 指定的文件夹内

先去搜索所需的模型，搜索的时候，加上关键词 GGUF（ollama 支持的模型文件格式）

- 国内大模型源文件下载地址 <https://modelscope.cn/models>
- 国外大模型源文件下载地址 <https://huggingface.co/models>

## Ollama 导入离线模型

Ollama 支持从 GGUF 文件导入模型，通过以下步骤来实现：

新建创建 Modelfile 文件

```bash
FROM ./Qwen2-0.5B.Q3_K_M.gguf
```

在 Ollama 中创建模型

```bash
ollama create mymodel -f Modelfile
```

