---
title: 参考
---

## OpenSSL 自签证书

=== "shell"

    ```
    openssl req -x509 -newkey rsa:2048 -nodes -days 365 \
    -keyout localhost.key \
    -out localhost.crt \
    -subj "/CN=localhost" \
    -addext "subjectAltName=DNS:localhost,IP:127.0.0.1,IP:::1"
    ```
=== "powershell"

    ```
    where.exe openssl
    ```

    ```
    $env:OPENSSL_CONF = "C:\Users\wjn\miniconda3\Library\ssl\openssl.cnf"
    ```

    ```
    openssl req -x509 -newkey rsa:2048 -nodes -days 365 `
    -keyout localhost.key `
    -out localhost.crt `
    -subj "/CN=localhost" `
    -addext "subjectAltName=DNS:localhost,IP:192.168.3.51,IP:::1"
    ```