---
title: nginx常用配置和命令
enlink: nginx-config-commands
date: 2018-02-08 09:19:09
categories: 运维
tags:
- nginx
---
## 命令
```bash
# 启动
nginx [-c /path/to/nginx.conf]
# 热更新配置
nginx -s reload
# 强制停止 nginx 服务
nginx -s stop
pkill nginx
kill -9 pid
# 优雅停止 nginx 服务, 等待请求结束后再停止
nginx -s quit
# 帮助
nginx -h
# 查看 nginx 版本
nginx -v
# 查看配置文件的正确性
nginx -t
# 粗诶配置文件, 并显示文件内容
nginx -T
# 测试配置文件, 但只显示错误信息
nginx -q
```

## 解析示例

### 静态文件

```yaml
location / {
    root /data/layui-soul-table/docs;
}

```

### 反向代理

```yaml
 server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  saodiyang.com;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                root /data/layui-soul-table/docs;
        }
        location /layui-soul-table-java/ {
                proxy_pass http://127.0.0.1:8080/;
                proxy_connect_timeout 600s;
                proxy_send_timeout 600s;
                proxy_read_timeout 600s;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        error_page 404 /404.html;
                location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
                location = /50x.html {
        }
}
```
## 启用https
### 1.购买免费证书
登录阿里云 -> 控制台 -> 安全（云盾） -> CA证书服务 -> 购买证书
![购买免费证书](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fr8xZ-Z1ylYPcoN1PCTk_5SDm6TM.png)

### 2.补全证书信息
点击补全，绑定域名

### 3.下载并配置
选择下载 证书for nginx
![下载证书](https://cdn.jsdelivr.net/gh/yelog/assets/images/FnMUOU4IZY2ZbMZcqggp4v7JHtej.png)

上面这个页面有相关的配置信息，下面简单介绍：

① 将下载文件中的 `*.pem`、`*.key`, 拷贝到 nginx 目录下 的 `cert` , 当然也可以是其他目录
② 修改 `nginx.conf`
```conf
server {
      listen       443 ssl;
      server_name  xiangzhangshugongyi.com;

      ssl_certificate      cert/214487958220243.pem;
      ssl_certificate_key  cert/214487958220243.key;

      ssl_session_cache    shared:SSL:1m;
      ssl_session_timeout  5m;

      ssl_ciphers  HIGH:!aNULL:!MD5;
      ssl_prefer_server_ciphers  on;

      location / {
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_set_header X-Forwarded-Proto https;
          proxy_redirect off;
          proxy_connect_timeout      240;
          proxy_send_timeout         240;
          proxy_read_timeout         240;
          # note, there is not SSL here! plain HTTP is used
          proxy_pass http://127.0.0.1:8080;
      }
  }
```
③ 重启 nginx，通过 证书绑定域名进行 https 访问到 服务器跑在 8080 的服务
![通过https访问](https://cdn.jsdelivr.net/gh/yelog/assets/images/FtTXB9QIYoZOlPFKsGg-ImxbL58N.png)

## 常用场景

### http 重定向到 https

```properties
server {
    listen 80;

}
```


