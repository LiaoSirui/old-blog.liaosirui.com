## Cronicle

Cronicle 的设计理念是将复杂的 Cron 任务管理变得简单直观，让用户能够专注于任务本身，而不是被繁琐的配置所困扰。

- Github 仓库：<https://github.com/jhuckaby/Cronicle>
- 文档：<https://cronicle.net/>

安装 Cronicle 的过程非常简便，只需几个简单的步骤即可完成。首先，确保您的服务器已经安装了 Node.js 和 npm（Node Package Manager），因为 Cronicle 依赖于这些环境来运行。接下来，通过以下命令下载并安装 Cronicle：

```bash
mkdir -p /opt/cronicle
cd /opt/cronicle

curl -L https://github.com/jhuckaby/Cronicle/archive/v0.9.77.tar.gz | tar zxvf - --strip-components 1
npm install
node bin/build.js dist
```

加入开机启动

```
/opt/cronicle/bin/control.sh start
```

## 参考资料

- <https://www.showapi.com/news/article/678859cb4ddd79f11a4cdcd2>