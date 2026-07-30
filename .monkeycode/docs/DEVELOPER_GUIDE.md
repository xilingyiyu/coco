# 开发者指南

## 项目目的

[xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) 是 [xiaozhi-esp32](https://github.com/xinnan-tech/xiaozhi-esp32-server) 开源硬件的后端服务系统，为智能终端设备提供语音交互、设备管理和外部服务集成能力。

**核心职责**:
- 通过 WebSocket 管理 ESP32 设备连接
- 处理语音交互链路（ASR → LLM → TTS）
- 执行意图函数（天气/快递/微信/点歌等）
- 提供 Web 管理界面（智控台）

## 环境搭建

### 前置条件

- Docker + Docker Compose
- 服务器（建议 2C2G+）
- 域名（可选，用于反向代理）

### 部署步骤

原始部署参考官方文档，核心步骤：

```bash
# 克隆项目
git clone https://github.com/xinnan-tech/xiaozhi-esp32-server.git

# 或使用本配置仓库
git clone https://github.com/xilingyiyu/coco.git
cd xiaozhi-esp32-server

# 启动全栈
docker compose -f docker-compose_all.yml up -d

# 或单独起服务端（无 web/redis/db 依赖）
docker compose -f docker-compose.yml up -d
```

## 配置文件结构

### 主配置 `.config.yaml`

位置：`./data/.config.yaml`（主机）→ `/opt/xiaozhi-esp32-server/data/.config.yaml`（容器）

```yaml
server:
  websocket: ws://47.108.153.232:8000/xiaozhi/v1/

selected_module:
  ASR: AliyunStreamASR
  LLM: OpenAILLM
  TTS: CosyVoiceSiliconflow
  VAD: SileroVAD
  Intent: function_call
  Memory: nomem

ASR:
  AliyunStreamASR:
    type: aliyun_stream
    appkey: <APPKEY>
    access_key_id: <AK>
    access_key_secret: <SK>

LLM:
  OpenAILLM:
    type: openai
    api_key: <API_KEY>
    base_url: https://api.deepseek.com
    model_name: deepseek-v4-flash

TTS:
  CosyVoiceSiliconflow:
    type: siliconflow
    model: FunAudioLLM/CosyVoice2-0.5B
    voice: FunAudioLLM/CosyVoice2-0.5B:alex
    private_voice: <VOICE_CLONE_URI>
    access_token: <TOKEN>

Intent:
  function_call:
    functions:
      - get_weather
      - wechat_send
      - kuaidi_query
      - play_music
      - web_search
      ...

prompt: "我是一个叫小智的台湾女孩，说话机车，声音好听，习惯简短表达。"
```

### 独立凭据文件

存储在 `./data/` 目录：

| 文件 | 内容 |
|------|------|
| `.wechat_webhook.json` | 企业微信 webhook URL |
| `.kuaidi_config.json` | 快递100 key + customer |
| `.yaohu_config.json` | 妖狐 API key |

## 插件开发

### 插件位置

```bash
./plugins_func/functions/*.py
```

### 插件模板

```python
from plugins_func.register import register_function, ToolType, ActionResponse, Action

FUNCTION_DESC = {
    "type": "function",
    "function": {
        "name": "my_function",
        "description": "函数描述，LLM 通过此描述决定何时调用",
        "parameters": {
            "type": "object",
            "properties": {
                "param1": {
                    "type": "string",
                    "description": "参数说明",
                },
            },
            "required": ["param1"],
        },
    },
}

@register_function("my_function", FUNCTION_DESC, ToolType.SYSTEM_CTL)
async def my_function(conn, param1: str = None):
    # 业务逻辑
    return ActionResponse(Action.REQLLM, "结果文本", None)
```

### Action 类型

| Action | 用途 | 说明 |
|--------|------|------|
| REQLLM | 调用 LLM 重述结果 | 将 result 发给 LLM 生成自然语言回复 |
| RESPONSE | 直接播报 | response 字段直接 TTS 播报，不走 LLM |
| RECORD | 记录到历史 | 记录工具调用但不回复 |
| NONE | 啥也不干 | - |
| ERROR | 错误 | - |

### 编辑/更新插件

```bash
# 写入插件
docker exec -i xiaozhi-esp32-server tee /opt/xiaozhi-esp32-server/plugins_func/functions/<name>.py < <plugin_content>

# 重启容器生效
docker restart xiaozhi-esp32-server
```

### Intent 函数注册

修改 `.config.yaml` 的 `Intent.function_call.functions` 列表后需重启：

```bash
docker restart xiaozhi-esp32-server
```

注意：`config_loader.py` 对 `list`/`str` 类型直接覆盖（非递归合并），因此 `functions` 必须完整列出所有函数名。

## 常用命令

```bash
# 查看容器状态
docker ps

# 查看服务日志
docker logs -f xiaozhi-esp32-server

# 进入容器
docker exec -it xiaozhi-esp32-server bash

# 重启服务
docker restart xiaozhi-esp32-server

# 测试 WebSocket 直连
echo "ws://47.108.153.232:8000/xiaozhi/v1/"
```

## nginx 配置

### 小智智控台反代

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

## 常见问题

### DeepSeek Key 失效

症状：设备回复"我很忙"等兜底消息。
解决：在 `.config.yaml` 更新 `LLM.OpenAILLM.api_key`，重启容器。

### 天气显示泸州而非叙永

原因：旧版用 `Action.REQLLM` 将天气文本发给 LLM 重述，LLM 自行加上了上级行政区划。
解决：新版已改为 `Action.RESPONSE` 直接 TTS 播报。

### 快递查询 601 错误

原因：快递100 key 未开通智能识别套餐。
解决：手动指定快递公司编码（`com` 参数）。

### 快递查询 408 错误

原因：顺丰/中通需要收件人手机号后四位。
解决：查快递时提供 `phone` 参数。

### 点歌失败（空结果）

原因：QQ 音乐对迅雷/付费歌曲返回空。
解决：自动降级到网易云音乐搜索。
