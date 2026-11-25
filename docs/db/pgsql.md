---
title: PostgreSQL
---
## 安装


=== "docker compose"

    === "13"
        ```
        # Use postgres/example user/password credentials
        version: '3.1'
        services:
            db:
                image: postgres:13
                restart: always
                network_mode: "host"
                environment:
                    POSTGRES_PASSWORD: Pgsql@2024
                    TZ: Asia/Shanghai
                volumes:
                    - ./data:/var/lib/postgresql/data
        ```

    === "18"
        ```
        # Use postgres/example user/password credentials
        version: '3.1'
        services:
            db:
                image: postgres:18
                restart: always
                network_mode: "host"
                environment:
                    POSTGRES_PASSWORD: Pgsql@2024
                    TZ: Asia/Shanghai
                volumes:
                    - ./data:/var/lib/postgresql
        ```