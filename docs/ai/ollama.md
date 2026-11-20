---
title: ollama
---

## [安装](https://ollama.com/download/linux){target="_blank"}

=== "Linux x86_64"
    
    迅雷下载 ```https://ollama.com/download/ollama-linux-amd64.tar.zst```

    ```bash
    sudo tar -xvf ollama-linux-amd64.tar.zst -C /usr/local/
    ```

    ```bash
    /usr/local/bin/ollama -v
    ```


配置环境变量

```
OLLAMA_HOST=0.0.0.0:11434
OLLAMA_KEEP_ALIVE=-1
```


## models

```
ollama run qwen3.5:4b --think=false
```


```bash
sudo tee /etc/systemd/system/ollama.service >/dev/null <<'EOF'
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=root
Group=root
Restart=always
RestartSec=3
Environment="OLLAMA_KEEP_ALIVE=-1"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ollama
sudo systemctl start ollama
```

```bash 
curl -s http://127.0.0.1:11434/api/version
```