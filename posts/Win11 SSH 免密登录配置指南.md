---
title: Win11 SSH 免密登录配置指南
date: 2026-02-07T21:38:00+08:00
categories: "技术分享"
id: "20260207"
cover: /images/posts/ssh_key.png
tags:
  - SSH
  - Linux
  - 2026
  - 运维
recommend: false
top: false
hide: false
author: "YEYUbaka"
description: 如何配置 SSH 密钥认证实现免密登录
showToc: true
---

:::note{type="success"}
最近在弄博客部署到Linux服务器时每次 SSH 登录都要输入密码很麻烦
本文教你如何配置 SSH 密钥认证，实现免密登录
:::

## 生成 SSH 密钥
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```
参数解析：
`-t rsa` :指定密钥类型为 RSA。这是目前最常用、兼容性最好的算法
`-b 4096`：指定密钥长度为 4096 位。数字越大越安全（默认是 2048，但我们直接拉满，安全感拉满！）。
`-f ~/.ssh/id_rsa`：指定生成的私钥文件保存路径和名字。这里会生成两个文件：
`~/.ssh/id_rsa`（私钥，千万别泄露！）
`~/.ssh/id_rsa.pub`（公钥，可以随便给）
`-N ""`：设置私钥的密码（passphrase）为空。

>如果你设了密码，每次用私钥时还得再输一次密码——那就违背“免密”初衷了！所以这里留空，直接回车也行。


## 上传公钥到服务器
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@your-server-ip
```

`ssh-copy-id`：专门将密钥添加到服务器的命令

`-i ~/.ssh/id_rsa.pub`：上传的**公钥文件路径**

`root@your-server-ip`：目标服务器的登录登录用户和服务器IP或者域名

## 测试登录
```bash
ssh root@your-server echo 'SSH 免密登录测试成功！' && whoami && hostname
```
如果看到类似输出*SSH 免密登录测试成功！*和账户名称主机名称即代表SSH免密登录设置成功！

```bash
SSH 免密登录测试成功！
root
my-awesome-server
```

## 提示！！！
- 权限很重要！
服务器上的 ~/.ssh 目录权限必须是 700，~/.ssh/authorized_keys 文件权限必须是 600。否则 SSH 出于安全考虑会直接拒绝使用密钥！
```bash
# （如果遇到权限问题）就在服务器上执行
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

- 注意私钥不要泄露！（私钥==密码）
把你的 id_rsa（私钥）文件安全地拷贝到新电脑的 ~/.ssh/ 目录下即可。但千万注意：私钥 = 密码，不要上传到 GitHub！

- 连接多台服务器？
  你可以把同一个公钥发给多台你需要连接的服务器上

  使用 `ssh-copy-id` 多跑几次就行，或者手动复制 `id_rsa.pub` 内容追加到各服务器的 `authorized_keys` 里

**参考资料**：

- [SSH的免密登录详细步骤（注释+命令+图）](https://blog.csdn.net/SXY16044314/article/details/90605069)
