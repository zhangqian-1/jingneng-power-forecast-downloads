# 京能七站功率预测 · 镜像下载

本仓库提供七站总功率预测服务的离线 Docker 镜像、下载说明和文件校验值。

- 发布版本：`v7station-2025-042dcd1`
- 模型版本：`trend_detail_7station_2025_v1`
- 镜像构建所用源码提交：`042dcd1ccd1dd75f22cf9849a1cfffc14a93802e`
- 平台：Linux AMD64、Linux ARM64

## 下载

[打开完整发布页面](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/tag/v7station-2025-042dcd1)

| 服务器架构 | 镜像包 | 大小 |
|---|---|---|
| `x86_64` / AMD64（Intel、AMD） | [offline-image-amd64-34703786520-1.zip](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-2025-042dcd1/offline-image-amd64-34703786520-1.zip) | 589.0 MiB |
| `aarch64` / ARM64 | [offline-image-arm64-34703803433-1.zip](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-2025-042dcd1/offline-image-arm64-34703803433-1.zip) | 530.1 MiB |
| ZIP 校验文件 | [SHA256SUMS](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-2025-042dcd1/SHA256SUMS) | 两个镜像 ZIP 的 SHA256 |

在目标 Linux 服务器执行 `uname -m` 查看架构，只需下载匹配架构的一个镜像 ZIP 和校验文件。GitHub 自动生成的 `Source code (zip)` / `Source code (tar.gz)` 仅包含本下载仓库的说明文件，不包含 Docker 镜像。

## 校验与部署

1. 将所选镜像 ZIP 和本发布页的 `SHA256SUMS` 放在同一个目录。在 Linux 中校验已下载的 ZIP：

   ```bash
   sha256sum --ignore-missing -c SHA256SUMS
   ```

   确认所选 ZIP 显示 `OK`。Windows PowerShell 可执行 `Get-FileHash -Algorithm SHA256 "镜像ZIP文件名"`，与 `SHA256SUMS` 中对应值比较。

2. 将镜像 ZIP 解压到一个单独的部署目录，进入该目录。此时使用的是 ZIP 内部的 `SHA256SUMS`，用于校验部署文件与 `image.tar.gz`：

   ```bash
   sha256sum -c SHA256SUMS
   docker load -i image.tar.gz
   ```

3. 保留包内 `.env` 中的镜像标签，按包内 `docs/Docker部署运行说明.md` 配置监听地址、端口和持久化目录。已安装并启动 Docker Engine、Docker Compose 2.20+ 后，在部署目录执行：

   ```bash
   docker compose config --quiet
   docker compose up -d --pull never --wait --wait-timeout 300
   docker compose ps
   ```

完整部署步骤见镜像 ZIP 内 `docs/Docker部署运行说明.md` 附录 A.2。接口与测点要求见同目录中的交接文档。默认服务仅监听服务器本机，跨机器接入按部署文档配置。

## 版本与文件内容

两个 ZIP 均复用同一版本原始构建产物，保持原文件名和字节内容。镜像包包含运行程序、模型权重、部署配置、文档和接口样例；无需重新训练模型。

容器运行需要平台通过接口提供七站真实历史。累计满足 672 个连续且温湿度可构造的历史点后，返回未来 24 小时的 96 点总功率预测。初始历史不足时返回 `409`，不会自动载入测试历史进行预热。

各架构的镜像 ID、构建提交、校验值和原构建验证记录以包内 `release.json` 为准。本次发布复用已完成验证的镜像，未重新训练或重新构建；目标环境的部署与平台接入仍需在实际服务器验收。
