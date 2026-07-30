# 接口文档

## WebSocket 协议

设备通过 WebSocket 长连接与服务端通信。

- **端点**: `ws://47.108.153.232:8000/xiaozhi/v1/`
- **协议**: 二进制音频流 + JSON 控制消息
- **认证**: MAC 地址绑定（设备白名单）

## HTTP API

主服务端口 8000 提供 HTTP 接口，智控台通过 nginx (xiaozhi.xilingyiyu.cn) 反代访问。

| 路径 | 方法 | 说明 |
|------|------|------|
| `/` | GET | 智控台 SPA 入口 |
| `/api/*` | - | 智控台 API |

OTA 接口端口 8003：

| 路径 | 方法 | 说明 |
|------|------|------|
| `/ota/*` | - | OTA 固件升级 |
| `/vision/*` | - | 视觉分析 |

## 插件函数列表

以下函数由 LLM 根据用户意图自动调用：

### 天气查询 `get_weather()`

| 属性 | 值 |
|------|-----|
| 描述 | 查询城市天气，不传参数自动显示叙永 |
| 参数 | `location`（可选，城市名） |
| 数据源 | [apihz.cn](https://www.apihz.cn) → [中国气象局](https://www.cma.gov.cn) |
| Action | RESPONSE（直接播报，不走 LLM） |
| 默认城市 | 叙永（四川） |

**输出示例**:
> 叙永当前天气，白天雷阵雨，夜间雷阵雨，气温28度，体感30度，最高31度，最低24度，湿度75%。明天雷阵雨，22到28度。后天雷阵雨，21到29度。

### 企业微信消息 `wechat_send()`

| 属性 | 值 |
|------|-----|
| 描述 | 通过[企业微信机器人](https://developer.work.weixin.qq.com/document/path/91770)发送消息 |
| 参数 | `content`（消息内容） |
| 配置 | `.wechat_webhook.json` 存 webhook URL |

### 企业微信设置 `wechat_set_webhook()`

| 属性 | 值 |
|------|-----|
| 描述 | 设置企业微信机器人 webhook URL |
| 参数 | `webhook_url`（webhook 地址） |

### 快递查询 `kuaidi_query()`

| 属性 | 值 |
|------|-----|
| 描述 | 查询快递物流信息 |
| 参数 | `num`（单号，必填），`com`（快递公司编码，可选），`phone`（手机号后四位，可选） |
| API | [快递100 poll/query.do](https://www.kuaidi100.com/openapi/) |
| 签名 | MD5(param + key + customer) → 32 位大写 |
| 配置 | `.kuaidi_config.json` |
| 轨迹 | 最多返回最近 3 条 |

**快递公司编码**:
| 编码 | 名称 |
|------|------|
| yuantong | 圆通 |
| zhongtong | 中通 |
| yunda | 韵达 |
| shentong | 申通 |
| shunfeng | 顺丰速运 |
| jd | 京东 |
| jtexpress | 极兔 |
| ems | EMS |
| debangwuliu | 德邦 |

### 快递绑定 `kuaidi_bind()`

| 属性 | 值 |
|------|-----|
| 描述 | 绑定快递100 API 账号 |
| 参数 | `key`（授权 key），`customer`（customer 编号） |

### 在线点歌 `play_music()`

| 属性 | 值 |
|------|-----|
| 描述 | 播放音乐，支持本地音乐和在线搜索 |
| 参数 | `song_name`（歌曲名，可含歌手；`"random"` 随机播放） |
| 搜索源 | QQ音乐优先 → 网易云降级（妖狐 API） |
| 播放方式 | 下载到 `tmpfs` (/tmp/yaohu_music) → TTS 音频文件播放 |
| 清理 | 播完自动清理 /tmp/yaohu_music |

### 网络搜索 `web_search()`

| 属性 | 值 |
|------|-----|
| 描述 | 搜索网络信息 |
| 参数 | - |

### 新闻获取 `get_news_from_newsnow()`

| 属性 | 值 |
|------|-----|
| 描述 | 获取最新新闻资讯 |
| 参数 | - |

### 新闻获取 `get_news_from_chinanews()`

| 属性 | 值 |
|------|-----|
| 描述 | 获取中国新闻网内容 |
| 参数 | - |

### 时间查询 `get_time()`

| 属性 | 值 |
|------|-----|
| 描述 | 查询当前时间 |
| 参数 | - |

### 角色切换 `change_role()`

| 属性 | 值 |
|------|-----|
| 描述 | 切换语音助手的角色设定 |
| 参数 | - |

## 插件 Action 接口

### ActionResponse

```python
class ActionResponse:
    action: Action   # 动作类型
    result: str      # 动作结果（传给 LLM 的文本）
    response: str    # 直接回复内容（Action.RESPONSE 时使用）
```

### Action 枚举

| 枚举 | code | 说明 |
|------|------|------|
| ERROR | -1 | 错误 |
| NOTFOUND | 0 | 没找到函数 |
| NONE | 1 | 啥也不干 |
| RESPONSE | 2 | 直接回复（TTS 播报 `response` 字段） |
| REQLLM | 3 | 调用 LLM 重述 `result` 后回复 |
| RECORD | 4 | 记录到对话历史，不播报 |

### 函数注册装饰器

```python
@register_function(name, func_desc, tool_type)
```

- `name`: 唯一函数名，与 Intent 配置对应
- `func_desc`: OpenAI function calling 格式描述
- `tool_type`: 工具类型（SYSTEM_CTL 等）
