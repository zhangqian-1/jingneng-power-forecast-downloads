# 京能七站功率预测镜像下载

当前交付版本：`v7station-single-step-a9614ff30661`。本仓库提供已构建、已测试的离线 Docker 镜像包，可直接下载后交给部署人员。

- 构建源码提交：`a9614ff306613133b25874c955003042a103d2b4`
- 模型：`single_step_7station_2025_v1`，已重新训练为过去288点输入、下一点输出。
- 接口：`POST /api/v1/fluxcast/compute`，平台适配版本 `fluxcast_v1`。

## 下载哪个文件

[打开新版发布页](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/tag/v7station-single-step-a9614ff30661)

| Linux 服务器架构 | 下载文件 | 大小 |
|---|---|---|
| `x86_64`，Intel / AMD | [AMD64 镜像包](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-single-step-a9614ff30661/offline-image-amd64-a9614ff30661-36149015922-1.zip) | 585.0 MiB |
| `aarch64`，ARM | [ARM64 镜像包](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-single-step-a9614ff30661/offline-image-arm64-a9614ff30661-36149081520-1.zip) | 525.8 MiB |
| ZIP 校验文件 | [SHA256SUMS](https://github.com/zhangqian-1/jingneng-power-forecast-downloads/releases/download/v7station-single-step-a9614ff30661/SHA256SUMS) | 校验上述两个 ZIP |

服务器执行 `uname -m` 查看架构，只需下载对应的一个镜像包和校验文件。发布页自动生成的 `Source code` 是本下载仓库的说明文件，不包含运行源码或镜像。完整工程见 [源码仓库](https://github.com/zhangqian-1/jingneng-power-forecast)；与本次镜像对应的版本为 [a9614ff](https://github.com/zhangqian-1/jingneng-power-forecast/tree/a9614ff306613133b25874c955003042a103d2b4)。

旧96点发布 `v7station-platform-11f034bc1833` 及更早版本保留供追溯，不能用于本次单点部署。

## 如何部署

目标服务器需要已安装并启动 Docker Engine、Docker Compose 2.20+。无需安装 Python、重新训练、重新构建或登录 GitHub 镜像仓库。

1. 将镜像 ZIP 和发布页的 `SHA256SUMS` 放在同一目录，执行 `sha256sum --ignore-missing -c SHA256SUMS`，确认所选 ZIP 显示 `OK`。Windows 可用 `Get-FileHash -Algorithm SHA256 "镜像ZIP文件名"` 与校验文件比较。
2. 将 ZIP 解压到独立部署目录，进入该目录，执行下面前两条命令。这里的 `SHA256SUMS` 为包内校验文件。
3. 保留 `.env` 中的镜像标签，按包内 `docs/Docker部署运行说明.md` 配置端口、监听地址及持久化目录，再执行启动命令。

```bash
sha256sum -c SHA256SUMS
docker load -i image.tar.gz
# 配置 .env 后启动
docker compose config --quiet
docker compose up -d --pull never --wait --wait-timeout 300
docker compose ps
```

包内包含镜像、`.env`、Compose 配置、交接文档、JSON 样例和 `release.json`。默认仅监听 `127.0.0.1:8000`，跨机器接入需配置内网地址或网关，具体见部署说明。

## 平台接入要点

- 每次提交 `point_table + frames` JSON，包含七站 35 个测点（19 个功率、8 个温度、8 个湿度）、96 个连续的 15 分钟历史点。测点编码以包内清单为准。
- 接口输入输出使用 UTC，模型内部转换为北京时间；平台不要重复换算。返回时间格式与请求一致。
- 累计达到 288 个连续且天气可用的历史点（3天）后，返回未来15分钟的1点总功率预测，`varname=totalPowerForecast`，`event_key=JNH.Fluxcast.Compute`，单位 MW。
- 功率缺失补 0；温湿度仅沿用过去真实值。历史或天气未就绪时返回 HTTP 200 和空 `result_point`，通过 `reason`、`message` 说明原因。
- 镜像不预装测试历史，需平台补传真实历史。旧版本升级使用新 runtime 目录并重新补传，不复用旧缓存。

## 已验证的内容

AMD64、ARM64 均已在 GitHub 完成容器启动、真实数据预测、重建后缓存恢复、73 天 / 7008 点滚动预测，以及镜像导出、重新导入和预测一致性测试。两种架构的历史测试集 MAPE 均约为 **3.1991%**，详细数值与镜像身份见包内 `release.json`。

下载附件与构建原件逐字节校验一致。测试报告见 [源码仓库对应 Release](https://github.com/zhangqian-1/jingneng-power-forecast/releases/tag/v7station-single-step-a9614ff30661)，本下载仓库仅提供镜像与校验文件。目标服务器的真实平台联调仍需完成，历史测试指标不代表未来实时预测精度。
