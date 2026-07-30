# 小智智控台 (xiaozhi-esp32-server) 配置仓库

[xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) 是为 ESP32 开源硬件提供后端服务的全栈系统。设备通过 WebSocket 连接服务端，实现语音交互（ASR → LLM → TTS）、设备管理和外部服务集成。

本仓库为 xiaozhi.xilingyiyu.cn 的部署配置与文档。

## 部署架构

| 容器 | 端口 | 职责 |
|------|------|------|
| xiaozhi-esp32-server | 8000(WS) / 8003(OTA) | Python 主服务 |
| xiaozhi-esp32-server-web | 8002 | 智控台 Vue SPA |
| xiaozhi-esp32-server-redis | 6379 | 缓存 |
| xiaozhi-esp32-server-db | 3306 | 数据库 |

## DNS 解析

域名通过 [阿里云 DNS](https://dns.console.aliyun.com) 解析：

| 域名 | 类型 | 目标 |
|------|------|------|
| xiaozhi.xilingyiyu.cn | CNAME → xilingyiyu.cn | nginx 反代到 8002 |
| video.xilingyiyu.cn | A → 106.55.151.24 | kvideo Next.js 服务 |

## 外部集成

| 服务 | 用途 | 文档 |
|------|------|------|
| [阿里云语音识别](https://help.aliyun.com/product/30412.html) | ASR 语音转文字 | AliyunStreamASR |
| [DeepSeek](https://platform.deepseek.com/) | LLM 语义理解 | deepseek-v4-flash |
| [硅基流动 SiliconFlow](https://siliconflow.cn/) | TTS 语音合成 | CosyVoice2-0.5B |
| [中国气象局](https://www.cma.gov.cn) (via [apihz.cn](https://www.apihz.cn)) | 天气数据 | tqyb.php API |
| [企业微信机器人](https://developer.work.weixin.qq.com/document/path/91770) | 群消息推送 | webhook 发送 |
| [快递100](https://www.kuaidi100.com/openapi/) | 物流轨迹查询 | poll/query.do |
| [妖狐 API](https://www.yaohud.cn) | QQ音乐/网易云搜索 | 音乐搜索下载 |

## 文档

| 文档 | 说明 |
|------|------|
| [项目索引](.monkeycode/docs/INDEX.md) | 文档导航与快速链接 |
| [架构设计](.monkeycode/docs/ARCHITECTURE.md) | 系统架构、技术栈、部署图 |
| [开发者指南](.monkeycode/docs/DEVELOPER_GUIDE.md) | 配置、插件开发、常见问题 |
| [接口文档](.monkeycode/docs/INTERFACES.md) | API 端点、WebSocket、插件列表 |

## 文档目录

```
.monkeycode/docs/
├── INDEX.md                    # 索引
├── ARCHITECTURE.md             # 架构
├── DEVELOPER_GUIDE.md          # 开发指南
├── INTERFACES.md               # 接口
└── 专有概念/                    # 核心概念
    ├── 设备.md
    ├── 插件.md
    ├── 意图链路.md
    └── 智控台Web.md
```
