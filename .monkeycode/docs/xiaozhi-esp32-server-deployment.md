# xiaozhi-esp32-server 部署文档

## 服务器信息

| 项目 | 值 |
|------|-----|
| IP | 47.108.153.232 |
| SSH | root / Dyhlovewhy525. |
| 域名 | xiaozhi.xilingyiyu.cn → 47.108.153.232 |
| 系统 | Ubuntu (Linux) |

## 容器栈 (Docker Compose)

4 个容器，均运行中：

| 容器 | 端口映射 |
|------|---------|
| xiaozhi-esp32-server | 8000, 8003 |
| xiaozhi-esp32-server-web | 8002 |
| xiaozhi-esp32-server-redis | 6379 (内部) |
| xiaozhi-esp32-server-db | 3306 (内部) |

## Nginx 反向代理

```
xiaozhi.xilingyiyu.cn:80  →  127.0.0.1:8002 (智控台 Web UI)
xiaozhi.xilingyiyu.cn:8000  →  docker-proxy 直通 (主服务 WebSocket + HTTP)
xiaozhi.xilingyiyu.cn:8003  →  docker-proxy 直通 (OTA + 视觉分析)
```

### Nginx 配置 (`/etc/nginx/sites-available/xiaozhi`)

```nginx
server {
    listen 80;
    server_name xiaozhi.xilingyiyu.cn;
    location / {
        proxy_pass http://127.0.0.1:8002;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 服务端点

| 端点 | 说明 |
|------|------|
| `xiaozhi.xilingyiyu.cn:8000` | 主服务 HTTP + WebSocket (`ws://.../xiaozhi/v1/`) |
| `xiaozhi.xilingyiyu.cn:8003/xiaozhi/ota/` | OTA 升级 |
| `xiaozhi.xilingyiyu.cn:8003/mcp/vision/explain` | 视觉分析 |
| `xiaozhi.xilingyiyu.cn:80` | 智控台 Web UI |

## 自定义配置

所有自定义配置文件位于容器内 `/opt/xiaozhi-esp32-server/data/`，编辑后需 `docker restart xiaozhi-esp32-server` 生效。

### `.config.yaml` — 核心配置

- ASR: 阿里云流式
- LLM: DeepSeek v4-flash
- TTS: EdgeTTS
- Intent 函数列表（设备可用工具）在 `Intent:` 块中声明

### 插件系统

插件在容器内 `/opt/xiaozhi-esp32-server/plugins_func/functions/`，宿主机编辑后通过 `docker exec -i xiaozhi-esp32-server tee` 写入容器，然后 `docker restart`。

| 插件 | 文件 | 功能 |
|------|------|------|
| 天气 | `get_weather.py` | 查询城市天气，默认叙永 |
| 点歌 | `play_music.py` | 本地播放 + 妖狐在线搜歌，下载到 `/tmp/yaohu_music`，播完清理 |
| 微信 | `wechat.py` | 企业微信 webhook 通知 |
| 快递 | `kuaidi.py` | 快递100 单号查询 |
| 新闻 | `get_news_from_newsnow.py` | 新闻聚合 |

### API 凭据

> 凭据已写入容器配置文件，此处仅列出用途，不包含真实值。

| 用途 | 配置方式 |
|------|---------|
| ASR / LLM / TTS | `.config.yaml` |
| 妖狐（点歌） | `.yaohu_config.json` |
| 天气 | 硬编码在 `get_weather.py` |
| 快递100 | `.kuaidi_config.json` |
| 企业微信 | `.wechat_webhook.json` |
| 智控台 | Web 登录 `xilingyiyu / Dyhlovewhy525.` |

### 配置合并规则

`.config.yaml` 使用递归合并：Mapping 类型递归合并，list/str 直接覆盖。自定义 `.config.yaml` 会合并到默认配置之上。

## 设备绑定

| 项目 | 值 |
|------|-----|
| MAC | 80:b5:4e:df:d1:b8 |
| 型号 | bread-compact-wifi-lcd |
| 智能体 ID | dc76f02dbc7444b186fef3da7e87ba4c |
| WebSocket 出口 IP | 111.55.146.29 |
| 默认城市（天气） | 四川省泸州市叙永县 |

## 可用语音指令

设备唤醒后可通过语音调用以下工具：

- `天气` — 叙永天气预报（默认）
- `杭州天气` — 指定城市天气
- `来首歌` / `点歌 周杰伦 七里香` — 在线搜歌播放
- `查快递` — 查询已绑定快递
- `发微信` — 企业微信通知
- `搜一下 xxx` — 网页搜索
- `换角色` — 切换 AI 角色

## OTA & 视觉

OTA 升级端点和视觉分析端点在 8003 端口，设备可通过 8003 端口进行固件升级和图像分析。
