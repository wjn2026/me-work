---
title: frp
---


内网穿透


[frp](https://github.com/fatedier/frp){target='_blank'}

* [frpc-desktop](https://github.com/luckjiawei/frpc-desktop)

* 配置

* [Server DashBoard](https://github.com/fatedier/frp?tab=readme-ov-file#server-dashboard)

=== "http"

    ```
    # The default value is 127.0.0.1. Change it to 0.0.0.0 when you want to access it from a public network.
    webServer.addr = "0.0.0.0"
    webServer.port = 7500
    # dashboard's username and password are both optional
    webServer.user = "admin"
    webServer.password = "admin"
    ```
=== "https"

    ```
    webServer.port = 7500
    # dashboard's username and password are both optional
    webServer.user = "admin"
    webServer.password = "admin"
    webServer.tls.certFile = "server.crt"
    webServer.tls.keyFile = "server.key"
    ```


* Systemd
frpc.service

```
[Unit]
Description=Frp Server
After=network.target

[Service]
Type=simple
ExecStart=/opt/frp_0.65.0_linux_amd64/frps -c frps.toml
Restart=on-failure
RestartSec=5s
User=frp
WorkingDirectory=/opt/frp_0.65.0_linux_amd64

[Install]
WantedBy=multi-user.target

```

* docker build

!!! note 

    注意使用docker 网络需要指定宿主机的ip

```
docker build -f .\dockerfiles\Dockerfile-for-frpc -t 192.168.3.12:5005/frpc:v0.66.0 .
```
=== "docker"

    ```
    docker run --rm -v ./frpc.toml:/frpc.toml 192.168.3.12:5005/frpc:v0.66.0 -c /frpc.toml
    ```

=== "docker compose"

    ```
    frpc:
        image: 192.168.3.12:5005/frpc:v0.66.0
        container_name: frpc
        volumes:
        - ./frpc.toml:/frpc.toml
        command: ["-c", "/frpc.toml"]
        restart: unless-stopped
    ```



```
serverAddr = "xxxx"
serverPort = 7000


[[proxies]]
name = "20-mysql"
type = "tcp"
localIP = "localhost"
localPort = 3306
remotePort = 22214

```


* Q&A

* login to server failed: EOF

查看frps中是否配置tcp_mux = false 若配置 frpc


---