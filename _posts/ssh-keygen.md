---
name: ssh代理配置
title: SSH代理配置
date: 2025-07-31
---

```bash
ssh-keygen -t rsa -b 4096 -C "fengyujie.jayden@jd.com"
ssh-keygen -t ed25519 -C "fengyujie.jayden@jd.com"
```



```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

在 `~/.ssh/config` 文件中添加配置（没有就新建一个）：

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

Host github.com
  User git
  Hostname ssh.github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
  Port 443
```

`pbcopy < ~/.ssh/id_ed25519.pub`
然后去 GitHub / GitLab / Gitee 等账号的 SSH Keys 设置中粘贴。

`ssh -T git@github.com` 