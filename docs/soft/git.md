---
title: Git
---

## 创建新分支

基于master分支创建并切换

```
git switch -c 新分支名 master
```

## 多账户 SSH 推送

同一台机器上用多个 GitHub 账号时，靠 `~/.ssh/config` 的 **Host 别名**区分身份：别名不同 → 使用不同私钥 → GitHub 认为是不同账号。仓库的 remote 地址也必须写这个别名，而不是 `github.com`。

### 1. 生成密钥（每个账号一把）

```shell
ssh-keygen -t ed25519 -C "wjn2026@github" -f C:/Users/wjn/.ssh/id_ed25519_wjn2026
ssh-keygen -t ed25519 -C "wjn0918@github" -f C:/Users/wjn/.ssh/id_ed25519_wjn0918
```

分别把生成的 `.pub` 公钥添加到对应 GitHub 账号的 **Settings → SSH and GPG keys**。

### 2. 配置 `~/.ssh/config`

Windows 下是 `C:\Users\wjn\.ssh\config`：

```ini
Host github-wjn2026
    HostName github.com
    User git
    IdentityFile C:/Users/wjn/.ssh/id_ed25519_wjn2026
    IdentitiesOnly yes

Host github-wjn0918
    HostName github.com
    User git
    IdentityFile C:/Users/wjn/.ssh/id_ed25519_wjn0918
    IdentitiesOnly yes
```

!!! tip "`IdentitiesOnly yes` 的作用"

    强制只使用 `IdentityFile` 指定的私钥，避免 ssh-agent 拿别的 key 先试，导致 GitHub 认成另一个账号。

### 3. 逐个验证

```shell
ssh -T git@github-wjn2026
ssh -T git@github-wjn0918
```

返回 `Hi <用户名>! You've successfully authenticated...`，核对其中的用户名是否就是该别名对应的账号。

### 4. 给仓库绑定身份

把 remote 里的 `github.com` 换成 config 中的 Host 别名，`:owner/repo` 部分保持不变：

```shell
git remote set-url origin git@github-wjn2026:wjn2026/me-reading.git
git remote -v
```

`git remote -v` 应显示 `git@github-wjn2026:wjn2026/me-reading.git`；若仍是 `github.com`，说明别名没生效。

!!! warning "提交身份是另一件事"

    推送身份由 SSH 别名决定，提交记录里的作者由 `user.name` / `user.email` 决定。多账号时建议在仓库内单独设置（不加 `--global`，只作用于当前仓库）：

    ```shell
    git config user.name "wjn2026"
    git config user.email "wjn2026@example.com"
    ```
