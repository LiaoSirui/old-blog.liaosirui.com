## UV

uv 是 Astral 公司推出的一款基于 Rust 编写的 Python 包管理工具，旨在成为 Python 的 Cargo

- <https://github.com/astral-sh/uv>

- <https://docs.astral.sh/uv/>

安装方式

```bash
pipx install uv
```

## 使用入门

创建一个项目，使用`uv init <project dir>`命令

```bash
uv init myproject
```

`uv init` 创建项目之后，会自动将项目使用`Git`来管理

同步项目依赖

```bash
uv sync
```

同步之后，会自动查找或下载合适的 `Python` 版本，创建并设置项目的虚拟环境，构建完整的依赖列表并写入

`uv.lock` 文件，最后将依赖同步到虚拟环境中。

使用`uv`管理项目的话，就使用`uv`的命令来运行代码，不要建议使用`python xxx.py`来运行。

```bash
uv run ./main.py
```

