# 小智智控台 (xiaozhi-esp32-server)

为 [xiaozhi-esp32](https://github.com/xinnan-tech/xiaozhi-esp32-server) 开源硬件提供后端服务的全栈系统。ESP32 设备通过 WebSocket 连接服务端，实现语音交互（ASR → LLM → TTS）、设备管理和外部服务集成。

## 部署架构

| 容器 | 端口 | 职责 |
|------|------|------|
| xiaozhi-esp32-server | 8000(WS) / 8003(OTA) | Python 主服务 |
| xiaozhi-esp32-server-web | 8002 | 智控台 Vue SPA |
| xiaozhi-esp32-server-redis | 6379 | 缓存 |
| xiaozhi-esp32-server-db | 3306 | 数据库 |

## 功能

- 天气查询（中国气象局数据源）
- 企业微信消息发送
- 快递物流查询（快递100）
- 在线点歌（QQ音乐 / 网易云）
- 网络搜索 / 新闻获取

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
