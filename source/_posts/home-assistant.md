---
title: 离线安装Home Assistant
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

最近搬到了新家，家里的智能设备也越来越多, 引入很多米家设备, 但博主使用的是苹果生态, 需要将这些不支持 `homekit` 的米家设备接入到 `homekit` 中, 经过调研发现 `Home Assistant` 是一个不错的选择, 本文会介绍联网安装的过程, 并且如果有需要联网的步骤, 也会提供**离线安装**的方法.

本篇主要介绍通过 `docker` 部署的方式

## 安装 Docker

如已有 `docker` 环境, 可以跳过这一步

以下以 Ubuntu 为例, 其他发行版请参考官方文档

```bash
# 1) 安装依赖并添加官方 GPG key
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 2) 添加 Docker 官方仓库
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# 3) 安装 Docker Engine
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4) 启动并验证
sudo systemctl status docker
# 如未启动, 可执行: sudo systemctl start docker
sudo docker run hello-world

```

可选: 免 `sudo` 使用 Docker

```bash
sudo usermod -aG docker $USER
# 重新登录后生效
docker ps
```

离线安装(无法访问官方仓库)

在另一台可联网机器上下载与你系统版本/架构对应的 `.deb` 包, 例如:

- `containerd.io_<version>_<arch>.deb`
- `docker-ce_<version>_<arch>.deb`
- `docker-ce-cli_<version>_<arch>.deb`
- `docker-buildx-plugin_<version>_<arch>.deb`
- `docker-compose-plugin_<version>_<arch>.deb`

拷贝到目标机器后安装:

```bash
sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
  ./docker-ce_<version>_<arch>.deb \
  ./docker-ce-cli_<version>_<arch>.deb \
  ./docker-buildx-plugin_<version>_<arch>.deb \
  ./docker-compose-plugin_<version>_<arch>.deb
# 如果提示缺少依赖, 需使用本地镜像/离线源补齐依赖后再重试
sudo systemctl status docker
# 如未启动, 可执行: sudo systemctl start docker
sudo docker run hello-world
```




