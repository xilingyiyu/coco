# 小智智控台 (xiaozhi-esp32-server) 文档

## 项目概述

小智智控台是 [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) 项目的 Web 管理界面和后端服务，由华南理工大学刘思源教授团队主导研发，专为 xiaozhi-esp32 开源硬件打造。系统通过 WebSocket 连接 ESP32 设备，提供语音交互（ASR → LLM → TTS → Intent）、设备管理、OTA 升级、视觉分析等能力。

## 访问入口

| 入口 | URL | 说明 |
|------|-----|------|
| 智控台 | https://xiaozhi.xilingyiyu.cn | Web 管理界面（反向代理到 8002） |
| WebSocket | ws://47.108.153.232:8000/xiaozhi/v1/ | 设备连接端点 |
| OTA/视觉 | 47.108.153.232:8003 | OTA 升级和视觉分析接口 |

## 核心文档

### [架构设计](./ARCHITECTURE.md)
系统架构、技术栈、容器部署结构、数据流和核心子系统。

### [开发者指南](./DEVELOPER_GUIDE.md)
环境搭建、开发工作流、常见任务和编码规范。

### [接口文档](./INTERFACES.md)
API 端点、WebSocket 协议、插件开发规范。

---

## 部署服务器

| 服务器 | 用途 |
|--------|------|
| 47.108.153.232 | 主服务器：xiaozhi-esp32-server 全栈（4 容器）+ nginx 反代 |
| 47.108.178.13 | 辅助服务器：cloudreve、视频、电影、timepost、人脸验证、mlzx、呓语云服务 |
| 106.55.151.24 | kvideo Docker 容器（呓语影视 Next.js） |

---

## 核心概念

| 概念 | 描述 |
|------|------|
| [设备](./专有概念/设备.md) | ESP32 硬件设备，通过 WebSocket 连接智控台 |
| [插件](./专有概念/插件.md) | Python 函数插件，扩展语音助手的能力 |
| [意图链路](./专有概念/意图链路.md) | 语音交互的处理流程：ASR → LLM → TTS → Intent |
| [智控台 Web](./专有概念/智控台Web.md) | Vue.js SPA 管理界面 |

## 模块文档

| 模块 | 描述 |
|------|------|
| [自研插件实现](./模块/自研插件实现.md) | 天气、微信、快递、点歌 4 个插件的详细设计和实现细节 |

---

## 快速链接

- [快速部署（从零开始）](./DEVELOPER_GUIDE.md#快速部署从零开始) — 从空白服务器到可用
- [功能清单](./ARCHITECTURE.md#功能清单) — 天气 / 快递 / 微信 / 点歌
- [插件列表](./INTERFACES.md#插件函数列表)
- [配置结构](./DEVELOPER_GUIDE.md#配置文件结构)
