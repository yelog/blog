---
title: 安装使用 Home Assistant
enlink: home-assistant
date: 2024-03-22 09:05:44
categories:
- 工具
- IOT
tags:
- home-assistant
- iot
---

## 前言

最近搬到了新家，家里的智能设备也越来越多, 引入很多米家设备, 但博主使用的是苹果生态, 需要将这些不支持 `homekit` 的米家设备接入到 `homekit` 中, 经过调研发现 `Home Assistant` 是一个不错的选择, 本文会介绍联网安装的过程

本篇主要介绍通过 `docker` 部署的方式

## 安装 Docker

如已有 `docker` 环境, 可以跳过这一步

以下以 Ubuntu 为例, 其他发行版请参考官方文档

```bash
# 1. 更新系统
sudo apt update
sudo apt upgrade -y

# 2. 安装依赖
sudo apt install -y ca-certificates curl gnupg lsb-release

# 3. 创建 keyrings 目录
sudo mkdir -p /etc/apt/keyrings

# 4. 添加 Docker GPG key（阿里云镜像）
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 5. 添加 Docker 阿里云源（推荐写法）
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 6. 更新索引
sudo apt update

# 7. 安装 Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 8. 启动并设为开机启动
sudo systemctl enable --now docker

# 9. 加入 docker 用户组
sudo usermod -aG docker $USER
```

## 使用 docker 安装启动 home assistant 

找到一个目录如 `/data/docker/admin`，创建文件 `docker-compose.yaml`，内容如下：
```yaml
version: '3'
services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant:2026.2.0b1
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
```

然后执行如下命令
```bash
# 启动 home assistant
docker compose up -d
# 查看 home assistant 日志
docker logs -f homeassistant
```

然后访问 ubuntu 的 `${ip}:8123` 即可进入 home assistant 的初始化界面。

> 如果有代理需要在容器内使用，可以参考如下添加环境变量的方式
```yaml
version: "3"
services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant:2026.2.0b1
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      # 代理（容器内生效，HACS/插件下载会走这里）
      HTTP_PROXY:  "http://192.168.1.2:7890"
      HTTPS_PROXY: "http://192.168.1.2:7890"
      ALL_PROXY:   "socks5://192.168.1.2:7891"

      # 不走代理的地址（强烈建议保留，避免局域网设备发现/本地服务变慢）
      NO_PROXY: "localhost,127.0.0.1,::1,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12,homeassistant.local"
```

## 安装 HACS

HACS 全程 Home Assistant Community Store,   是 Home Assistant 的一个第三方插件商店, 可以方便地安装各种插件和主题。

安装命令非常简单，只要如下一个命令即可
```bash
wget -O - https://get.hacs.xyz | bash -
```

等待安装完成后，重启 Home Assistant 容器
```bash
docker restart homeassistant
```

接下来需要在 Home Assistant 界面中启用 HACS，具体步骤如下: 打开 Home Assistant 界面，依次点击 设置 -> 设备与服务 -> 集成 -> 添加集成 -> 搜索 HACS 并添加

HACS 会弹出一个 GITHUB 授权页面，复制机器码到浏览器中授权

刷新 Home Assistant 页面后，左侧菜单出现了 `HACS` 选项，表示安装成功

## 安装并配置 Xiaomi Miio

在 HACS 中搜索 `Xiaomi Miio`，点击下载，然后到 `设置 -> 设备与服务 -> 集成` 中添加 `Xiaomi Miio` 集成

登录米家账号，选择需要添加的设备即可




