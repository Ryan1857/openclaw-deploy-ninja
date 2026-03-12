# OpenClaw Deploy Ninja

一个面向 OpenClaw 的一键安装脚本项目，目标是把「安装 + 安全加固 + 渠道配置 + Agent 绑定」一次走完。

## 项目亮点

1. 安全优先：默认仅主 Agent 保留宿主机操作能力，子 Agent 运行在 Docker 沙箱，避免 OpenClaw 裸奔。
2. 自动安装 `openclaw-china/channels` 插件，开箱即用中国区渠道能力。
3. 向导式配置渠道：支持 Telegram、钉钉、飞书（中国版）、企业微信。
4. 向导式 Agent 配置：支持主 Agent + 多子 Agent，并支持渠道多账号绑定。
5. 双安装模式：支持联网安装和本地离线安装，兼顾可翻墙与不可翻墙场景。

## 支持平台

- macOS (Intel / Apple Silicon)
- Linux (x64)

## 快速开始

```bash
chmod +x openclaw-install.sh
./openclaw-install.sh
```

脚本执行后会进入交互式向导，依次完成：

- 安装模式选择（联网 / 本地）
- 模型区域选择（国外 / 国内）
- OpenClaw 安装与 Onboard
- 插件与依赖安装
- 主 Agent 与子 Agent 配置
- 渠道账号录入与绑定
- 状态检查与控制台打开

## 两种安装模式

### 1) 联网安装

适用于可访问外网环境。脚本会自动安装或升级依赖并安装最新 OpenClaw。

### 2) 本地离线安装

适用于受限网络环境。请提前将离线包放到项目根目录下的 `offline-packages/`。

必需文件名模式如下：

- `openclaw-*.tgz`
- `clawhub-*.tgz`
- `openclaw-china-channels-*.tgz`
- `node-v*-<os>-<arch>.tar.gz` 或 `node-v*-<os>-<arch>.tar.xz`
- macOS: `Docker-arm64.dmg` 或 `Docker-amd64.dmg`
- Linux: `get-docker.sh`

> 建议将离线包按上述命名放入 `offline-packages/`，脚本会自动识别并校验。

## 安全模型说明

- 主 Agent (`main`)：负责总体编排与管理。
- 子 Agent：默认被限制在沙箱策略内，避免直接获得宿主机高风险操作能力。
- 可执行型子 Agent：执行能力仍绑定在 Docker 沙箱中，并启用命令白名单与工作区文件访问限制。

## 渠道与账号绑定

向导会引导你为每个 Agent 选择管理渠道，并支持：

- Telegram
- 钉钉
- 企业微信（Webhook 或应用模式）
- 飞书中国版

支持同一渠道多账号配置，并可按 `channel:accountId` 进行精细绑定。

## 卸载

```bash
./openclaw-install.sh uninstall
```

卸载会清理 OpenClaw 数据目录、npm 全局包以及脚本创建的相关沙箱容器。

## 常用目录

- 安装根目录：`~/.openclaw`
- 配置目录：`~/.openclaw/conf`
- 本地 npm 前缀：`~/.openclaw/npm-global`

## 免责声明

本项目用于自动化部署与配置辅助。请在使用前确认你的渠道账号、API 密钥与企业合规要求，并在生产环境先做灰度验证。
