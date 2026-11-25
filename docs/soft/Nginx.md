---
title: Nginx
---

## 安装



=== "docker-compose"

    ```
    services:
    nginx-1.20.2:
        image: nginx:1.20.2-alpine
        volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
        - ./conf.d:/etc/nginx/conf.d
        network_mode: "host"
    ```

=== "docker"

    ```
    FROM nginx:1.24.0
    # 设置工作目录，可选
    WORKDIR /app

    # 将你的 Java 应用程序 JAR 文件添加到容器中
    COPY dist/ /app/dist/

    # 启动 Nginx
    CMD ["nginx", "-g", "daemon off;"]

    # mkdir -p school-front/conf.d
    ```

=== "手动"


    * PCRE（Perl Compatible Regular Expressions）库，而 Nginx 的 http_rewrite_module 依赖这个库来处理正则表达式
    sudo apt install libpcre3 libpcre3-dev

    sudo apt install zlib1g-dev  # 安装 zlib 开发库

    1. [下载安装包](http://nginx.org/)
    3. 安装相关依赖

    ```
    yum -y install gcc pcre-devel openssl openssl-devel
    ```
    4. 编译nginx

    ```
    ./configure --prefix=解压到的文件目录（安装目录，不需要创建）  \
    --without-http_rewrite_module \
    --without-http_gzip_module
    ```
    5. 安装
    ```
    make install
    ```
    6. 创建软连接（可选）

    ln  -s  [源文件或目录]  [目标文件或目录]

    例:

        ln -s /usr/local/nginx/nginx1.6.3/sbin/nginx /usr/local/bin/nginx

=== "Ubuntu"

    ```
    sudo apt install nginx
    ```


## 配置

### 返回字符串

```
server {
    listen 8888;

    location /work_info {
    default_type application/json;
    return 200 '{"msg":"请访问 http://192.168.3.12:8008"}';
    }

}

```

### 重定向

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name localhost;

    location /work_info {
        return 301 http://192.168.3.12:8008;
    }
}
```

### 授权应用到期

!!! note "逻辑"

    通过判断是否存在一个文件 来判断是否授权

```
server {
    listen 8888;

    location / {

        if (!-f /etc/nginx/license.valid) {
            add_header Content-Type "text/plain; charset=utf-8";
            return 200 "已于 2026-01-01 到期，请联系管理员";
        }

        root /packages/resources/;
        autoindex on;
    }
}

```

* crontab 脚本 到期删除

```
vi /usr/local/bin/check_license.sh
```

```
#!/bin/bash
EXPIRE_DATE="2026-01-01"

if [ "$(date +%Y-%m-%d)" \> "$EXPIRE_DATE" ]; then
    echo "到期时间： $EXPIRE_DATE"
    rm -f /etc/nginx/license.valid
    # docker exec -it 容器名 rm -rf /etc/nginx/license.valid
else
    echo "未到期，到期时间：$EXPIRE_DATE"
fi

```

```
crontab -e
0 * * * * /usr/local/bin/check_license.sh
```


```
docker exec 容器名 sh -c 'echo "2026-04-08" > /etc/nginx/license.valid'
```



## 案例

### basic认证
1. 生成一个账户密码文件 pass_file

```
htpasswd -c -d /etc/nginx/conf.d/pass_file fd
```

fd ---用户名

2. nginx.conf

```
auth_basic "need user and password";
auth_basic_user_file /etc/nginx/conf.d/pass_file;
```

实例
```
location / {
    auth_basic "need user and password";
    auth_basic_user_file /etc/nginx/conf.d/pass_file;
    root   /usr/share/nginx/html;
    index  index.html index.htm;
}
```

### Xinference

```
server {
    listen 8000;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:9997;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### mysql

!!! warning

    这段 不能 放在 http {} 里
    一般直接放在 /etc/nginx/nginx.conf 末尾

```
stream {
    upstream mysql_backend {
        server 127.0.0.1:22214;   # 本地 MySQL / frp 映射端口
    }

    server {
        listen 443 ssl;
        proxy_pass mysql_backend;

        ssl_certificate     /etc/letsencrypt/live/frp.catpd.cn/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/frp.catpd.cn/privkey.pem;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_session_timeout 10m;
    }
}

```

### 静态页面

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   C:/Users/wjn/github/xiaozhi-esp32-server/main/xiaozhi-server/test;
            index  test_page.html index.html index.htm;
        }

      
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

      
    }


}

```

### 反向代理

```
server {
    listen 443 ssl;
    server_name registry.example.com;

    ssl_certificate /root/certs/fullchain.pem;
    ssl_certificate_key /root/certs/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 0;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # docker registry 需要以下配置
        proxy_read_timeout 900;
    }
}
```


### https


```
server {
    listen 443 ssl;
    server_name registry.example.com;

    ssl_certificate /root/certs/fullchain.pem;
    ssl_certificate_key /root/certs/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 0;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # docker registry 需要以下配置
        proxy_read_timeout 900;
    }
}

# 可选：HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name registry.example.com;
    return 301 https://$host$request_uri;
}

```

查看证书日期

```
openssl x509 -in xx.pem -noout -dates
```

### 代理资源

!!! note 

    在 server 块中添加     
    absolute_redirect off;
    port_in_redirect off;
    让 Nginx 在生成重定向 URL 时不带端口号

```
location ^~ /resources {
    alias /opt/resources;
    autoindex on;
    charset utf-8;
}
```

```
server {
    listen   10006;
    server_name  127.0.0.1;
    location / {
      alias /opt/resources/;
      autoindex on;
      charset utf-8;
    }
}
```

```

server {
    listen 88;
    location ^~/jw {
        auth_basic "need user and password";
        auth_basic_user_file /etc/nginx/basic/jw;
        alias /data/share/resources/jw;
        autoindex on;
        charset utf-8;
    }
}
```

### 代理openclaw
```
server {
   listen 80;
   server_name openclaw.example.com;
   return 301 https://$host$request_uri;
}
server {
   listen 443 ssl http2;
   server_name openclaw.example.com;
   ssl_certificate /etc/nginx/ssl/openclaw.crt;
   ssl_certificate_key /etc/nginx/ssl/openclaw.key;
   location / {
       proxy_pass http://127.0.0.1:3000; # OpenClaw运行端口
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       # WebSocket支持
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_read_timeout 300s;
   }
}
```


```
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "trustedProxies": ["127.0.0.1"],
    "auth": {
      "mode": "trusted-proxy",
      "trustedProxy": {
        "userHeader": "x-forwarded-user",
        "requiredHeaders": ["x-forwarded-proto", "x-forwarded-host"]
      }
    },
    "controlUi": {
      "allowedOrigins": ["https://agent.lly1997.top"]
    }
  }
}

```