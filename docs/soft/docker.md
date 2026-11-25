---
title: Docker
---


## 安装

=== "Ubuntu"

    ```shell
    # step 1: 安装必要的一些系统工具
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg

    # step 2: 信任 Docker 的 GPG 公钥
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Step 3: 写入软件源信息
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Step 4: 安装Docker
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

=== "Binaries安装"

    https://docs.docker.com/engine/install/binaries/#install-static-binaries

    ```shell
    wget https://download.docker.com/linux/static/stable/x86_64/docker-26.1.4.tgz
    tar -xzvf docker-26.1.4.tgz
    sudo cp docker/* /usr/bin/
    ```

    `sudo vi /etc/systemd/system/docker.service`

    ```ini
    [Unit]
    Description=Docker Application Container Engine
    Documentation=https://docs.docker.com
    # BindsTo=containerd.service
    # After=network-online.target firewalld.service containerd.service
    After=network-online.target firewalld.service
    Wants=network-online.target
    # Requires=docker.socket

    [Service]
    Type=notify
    # the default is not to use systemd for cgroups because the delegate issues still
    # exists and systemd currently does not support the cgroup feature set required
    # for containers run by docker
    # ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    ExecStart=/usr/bin/dockerd
    ExecReload=/bin/kill -s HUP $MAINPID
    TimeoutSec=0
    RestartSec=2
    Restart=always

    # Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
    # Both the old, and new location are accepted by systemd 229 and up, so using the old location
    # to make them work for either version of systemd.
    StartLimitBurst=3

    # Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
    # Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
    # this option work for either version of systemd.
    StartLimitInterval=60s

    # Having non-zero Limit*s causes performance problems due to accounting overhead
    # in the kernel. We recommend using cgroups to do container-local accounting.
    LimitNOFILE=infinity
    LimitNPROC=infinity
    LimitCORE=infinity

    # Comment TasksMax if your systemd version does not support it.
    # Only systemd 226 and above support this option.
    # TasksMax=infinity

    # set delegate yes so that systemd does not reset the cgroups of docker containers
    Delegate=yes

    # kill only the docker process, not all processes in the cgroup
    KillMode=process

    [Install]
    WantedBy=multi-user.target
    ```

    ```shell
    systemctl daemon-reload
    ```

    1. docker-compose

    https://docs.docker.com/compose/install/linux/#install-the-plugin-manually

    ```shell
    DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
    mkdir -p $DOCKER_CONFIG/cli-plugins
    curl -SL https://github.com/docker/compose/releases/download/v2.32.0/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose

    chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
    ```


## 删除none 镜像

docker image prune -f

## 镜像发布到阿里云


1. 登录阿里云Docker Registry
```
docker login --username=wjn_0918 registry.cn-hangzhou.aliyuncs.com
```
2. 从Registry中拉取镜像
```
docker pull registry.cn-hangzhou.aliyuncs.com/wjn0918/soft:[镜像版本号]
```
3. 将镜像推送到Registry
```
docker login --username=wjn_0918 registry.cn-hangzhou.aliyuncs.com
```
```
docker tag eb175b0743cc registry.cn-hangzhou.aliyuncs.com/wjn0918/soft:mysql-5.7.39
```
```
docker push registry.cn-hangzhou.aliyuncs.com/wjn0918/soft:[镜像版本号]
```

## 强制拉取镜像

docker pull --platform=linux/arm64 arm64v8/postgres:13

## Install


=== "[Ubuntu](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.576a1b11xhStgi)"

```
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# step 2: 信任 Docker 的 GPG 公钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Step 3: 写入软件源信息
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 4: 安装Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Docker Compose 

同一个 docker-compose.yml 里的所有服务自动在 同一个网络（默认网络）里

* 查看某个网络里有哪些容器
```
docker inspect network prometheus_default
```
### 指定配置

docker-compose -f <配置文件路径> up -d

### tag multiple services

=== "powershell"

    tag-and-push.ps1
    ```
    # Set your target registry
    $targetRegistry = "myregistry.azurecr.io"

    # Get all images from docker-compose
    $images = docker compose config --images

    # Tag and push each image
    foreach ($image in $images) {
        $newTag = "$targetRegistry/$image"
        Write-Host "Tagging: $image -> $newTag"
        docker tag $image $newTag
        
        Write-Host "Pushing: $newTag"
        docker push $newTag
    }
    ```

### run one service

```
docker compose up -d <service_name>
```

## build

```
docker build -f Dockerfile.prod -t myapp:latest .
```

### build 案例

!!! note "buid 规则"

    Dockerfile 里：只有「最后一个 FROM 开始的那个 stage」会成为最终镜像。

    规则 1️⃣：每个 FROM 都是一个「新世界」
    规则 2️⃣：只有「最后一个 FROM」决定最终镜像
    规则 3️⃣：只有你“亲手 COPY 过来”的才会进来

```
FROM node:22 AS web-builder

WORKDIR /web/frpc
COPY web/frpc/ ./
RUN npm install
RUN npm run build

FROM golang:1.24 AS building

COPY . /building
COPY --from=web-builder /web/frpc/dist /building/web/frpc/dist
WORKDIR /building

RUN env CGO_ENABLED=0 go build -trimpath -ldflags "-s -w" -tags frpc -o bin/frpc ./cmd/frpc

FROM alpine:3

RUN apk add --no-cache tzdata

COPY --from=building /building/bin/frpc /usr/bin/frpc

ENTRYPOINT ["/usr/bin/frpc"]
```


## 进入容器

```
docker run -it --rm \
--entrypoint /bin/bash \
school-etl
```


## 执行sql

```
docker exec -i mysql sh -c 'exec mysql -uroot -pMysql@2024 ' <<C 
select 1 
C

```

```
docker exec -i mysql8 sh -c 'exec mysql -uroot -pMysql@2023 ' <<C 
use report_conf; show tables;
C
```

## 执行shell

```
docker exec 容器名 sh -c 'echo "2026-04-08" > /etc/nginx/license.valid'
```

```
docker exec  容器名 sh -c 'rm -rf /etc/nginx/license.valid'
```

打标签

docker tag eb175b0743cc registry.cn-hangzhou.aliyuncs.com/wjn0918/soft:mysql-5.7.39





## 国内镜像

```
vi /etc/docker/daemon.json
```
- 新增
```
{"registry-mirrors": ["https://dockerproxy.cn"]}
```
- 重载systemd管理守护进程配置文件

```
sudo systemctl daemon-reload
```

- 重启服务

```
sudo systemctl restart docker
```



## migrate docker data directory

- Stop the Docker daemon.

```
systemctl stop docker
```

- Move the existing Docker data directory to the new location.

```
cp -a /usr/local/docker/. /home/docker/
```

- Start the Docker daemon with the --data-root option pointing to the new location.

```
vi /etc/docker/daemon.json
```
```
{
  "data-root": "/home/docker"
}
```




## 所有用户使用

groupadd docker
usermod -aG docker shingi
newgrp docker


## 常见问题

### 自建仓库

* 打标签

```
docker tag 63e8ae20eaef 192.168.3.12:5005/mysql:8.4.8
```
* 推送
```
docker push 192.168.3.12:5005/mysql:8.4.8
```

* Error response from daemon: Get "https://192.168.3.12:5005/v2/": http: server gave HTTP response to HTTPS client

=== "linux"

    编辑 Docker 配置文件

    ```
    vi /etc/docker/daemon.json
    ```

    如果文件不存在就新建，内容如下：

    ```
    {
    "insecure-registries": ["192.168.3.12:5005"]
    }
    ```

    ```
    systemctl daemon-reexec
    systemctl restart docker
    ```

=== "docker desktop"

![img](../../../assets/imgs/work/Soft/Docker/dockerdesktop_setting.png)