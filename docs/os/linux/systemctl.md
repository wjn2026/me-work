---
title: Systemctl服务注册
---

## nginx

cp nginx.service /usr/lib/systemd/system/
systemctl daemon-reload
systemctl enable nginx.service
systemctl start nginx.service

```
[Unit] 
Description=nginx - high performance web server 
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/data/nginx/logs/nginx.pid
LimitNOFILE=100000
LimitNPROC=65535
LimitMEMLOCK=infinity
ExecStartPre=/opt/third/nginx/sbin/nginx -t -c /opt/third/nginx/conf/nginx.conf
ExecStart=/opt/third/nginx/sbin/nginx -c /opt/third/nginx/conf/nginx.conf
ExecReload=/opt/third/nginx/sbin/nginx -s reload
ExecStop=/opt/third/nginx/sbin/nginx -s stop
PrivateTmp=true
Restart=on-failure
RestartSec=40

[Install]
WantedBy=multi-user.target     
```