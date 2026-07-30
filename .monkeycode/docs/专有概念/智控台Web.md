# 智控台 Web

## 什么是智控台 Web？

基于 Vue.js 2.x 的单页应用，提供 xiaozhi-esp32-server 的 Web 管理界面。包含设备管理、配置修改、日志查看等功能。

## 部署方式

- 容器：xiaozhi-esp32-server-web
- 镜像：ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
- 端口：8002
- 访问：https://xiaozhi.xilingyiyu.cn（nginx 反代）

## 技术栈

- Vue.js 2.x
- webpack 构建
- REST API 与后端通信

## nginx 反向代理

```nginx
server {
    listen 80;
    server_name xiaozhi.xilingyiyu.cn;
    location / {
        proxy_pass http://127.0.0.1:8002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
