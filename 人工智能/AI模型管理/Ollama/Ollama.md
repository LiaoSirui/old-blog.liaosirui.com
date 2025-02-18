## Ollama

一站式 AI 模型管理与交互平台

Ollama 是一个开源的大型语言模型服务工具，旨在帮助用户快速在本地运行大模型。通过简单的安装指令，用户可以通过一条命令轻松启动和运行开源的大型语言模型。 它提供了一个简洁易用的命令行界面和服务器，专为构建大型语言模型应用而设计。用户可以轻松下载、运行和管理各种开源 LLM。与传统 LLM 需要复杂配置和强大硬件不同，Ollama 能够让用户在消费级的 PC 上体验 LLM 的强大功能。

Ollama 会自动监测本地计算资源，如有 GPU 的条件，会优先使用 GPU 的资源，同时模型的推理速度也更快。如果没有 GPU 条件，直接使用 CPU 资源。

- GitHub：<https://github.com/ollama/ollama>

- 支持的模型：<https://ollama.com/library>

## Ollama 常用命令

| 命令            | 描述                   |
| --------------- | ---------------------- |
| `ollama serve`  | 启动 Ollama            |
| `ollama create` | 从 Modelfile 创建模型  |
| `ollama show`   | 显示模型信息           |
| `ollama run`    | 运行模型               |
| `ollama stop`   | 停止正在运行的模型     |
| `ollama pull`   | 从注册表中拉取模型     |
| `ollama push`   | 将模型推送到注册表     |
| `ollama list`   | 列出所有模型           |
| `ollama ps`     | 列出正在运行的模型     |
| `ollama cp`     | 复制模型               |
| `ollama rm`     | 删除模型               |
| `ollama help`   | 显示任意命令的帮助信息 |