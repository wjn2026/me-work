---
title: Linux
---

## 查看服务是否配置自启

```bash
systemctl is-enabled xxx.service
```

## 创建 sudo 用户

!!! note "管理员组名不同"

    CentOS / RHEL / Fedora 这类系统的管理员组叫 `wheel`，不叫 `sudo`。

=== "Debian / Ubuntu"

    ```shell
    adduser wjn
    usermod -aG sudo wjn
    ```

=== "CentOS / RHEL / Fedora"

    ```shell
    useradd -m wjn
    usermod -aG wheel wjn
    ```

查看认证日志：

=== "Debian / Ubuntu"

    ```shell
    sudo tail -f /var/log/auth.log
    ```

=== "CentOS / RHEL / Fedora"

    ```shell
    sudo tail -f /var/log/secure
    ```

## 公钥登录

### 服务器开启公钥认证

编辑 `/etc/ssh/sshd_config`：

```shell
sudo vim /etc/ssh/sshd_config
```

```conf
PubkeyAuthentication yes
```

### 客户端生成密钥

```shell
ssh-keygen -t rsa -C "wjn_0918@163.com" -f C:/Users/wjn/.ssh/id_rsa_gx
```

### 服务器添加公钥

```shell
echo "xxxxx" >> ~/.ssh/authorized_keys \
  && chmod 600 ~/.ssh/authorized_keys
```

### 登录

```shell
ssh -i 私钥地址 user1@192.168.12.30
```

登录失败时打开调试日志：

```shell
ssh -vvv -i 私钥地址 user1@192.168.12.30
```

## SSH 配置

### 配置

```conf
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

### 测试

修改后先检查语法，再重载服务：

```shell
sudo sshd -t
sudo systemctl reload sshd
```

确认最终生效的值：

```shell
sudo sshd -T | grep -iE 'pubkeyauthentication|passwordauthentication|permitrootlogin'
```

### 生效值不对时排查覆盖源

`sshd_config.d/` 下的文件会覆盖主配置，按文件来源全局搜一遍：

```shell
sudo grep -RiE 'PasswordAuthentication|PermitRootLogin|PubkeyAuthentication' \
  /etc/ssh/sshd_config /etc/ssh/sshd_config.d/ 2>/dev/null
```
