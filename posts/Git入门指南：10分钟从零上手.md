---
title: Git入门指南：10分钟从零上手
date: 2026-02-22T08:58:56+08:00
categories: "学习笔记"
id: "20260222"
cover: /images/posts/git.png
tags:
  - Git
  - 教程
  - 2026
recommend: false
top: false
hide: false
author: "YEYUbaka"
description: Git 太难？10分钟就够了！本指南专为零基础新手设计，跳过复杂理论，直击核心命令，让你快速掌握版本控制精髓。
showToc: true
---

:::note{type="success"}
从安装配置到提交推送，本文用最简路径讲解 Git 必备操作。无需编程背景，跟着步骤实操，轻松开启高效协作之旅，适合想快速上手的开发者。
:::

**参考资料**:
- [Git Bash详细教程](https://blog.csdn.net/qq_36667170/article/details/79085301)
- [Git中文文档](https://git-scm.cn/doc)


## Git 超极简入门指南：10分钟从零到上手

### 一、了解Git

#### 1. 什么是 Git？

简单来说，Git 是一个**分布式版本控制系统**。

- 版本控制：能记录文件的每一次改动。写错了？没关系，一键回退到上一个的版本。

- 分布式：每个人的电脑上都有完整的版本库，断网也能工作，同时保证了多个成员的并行作业。

#### 2.为什么需要Git？

| 场景 | 没有 Git | 有 Git |
| :------------: | :------------: | :------------: |
| 代码写错 | 手动备份，容易丢失 | 一键回退到任意版本 |
| 多人协作 | 文件覆盖，冲突频发 | 分支管理，合并审查 |
| 追溯历史 | 靠记忆或文档 | 完整的提交记录 |


### 二、 下载安装与配置
#### 1. 访问Git官网下载对应安装包
[点这里 选择对应系统下载](https://git-scm.cn/install/ "点这里 选择对应系统下载")

这里我选择我系统(Win11 x64)对应版本

[![](/images/posts/git-guide/git-installer.png)](/images/posts/git-guide/git-installer.png)

安装完成后，打开终端（Windows 下是 Git Bash，Mac/Linux 是 Terminal）
#### 2. 初始配置
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
```
如下图一样list中能看到配置的用户名和邮箱就代表配置成功了！

[![](/images/posts/git-guide/git-config.png)](/images/posts/git-guide/git-config.png)

>这样Git就知道"是谁提交了代码"

### 三、Git的流程
>在了解对代码操作的命令前我们需要先知道代码是怎么从本地设备上到远程存储代码的设备(远程仓库)上的

```
工作区 (Workspace)				你实际修改文件的地方
    ↓ git add
暂存区 (Index/Stage)			确认改动的代码暂存处
    ↓ git commit
本地仓库 (Local Repository)		项目的所有版本历史记录
    ↓ git push
远程仓库 (Remote)			云端保存的项目版本（如 GitHub/Gitee）

```
### 四、常用命令实操
>这里我就以上传我博客的md文章为例
#### 1. 创建仓库

```bash
# 方式一：从零开始
git init #初始仓库时无论本地文件夹内是否为空都可以

# 方式二：克隆现有项目
git clone https://github.com/用户名/项目名.git
```
在博客文章文件夹内右键`Open Git Bash here`
执行`git init`后会在文件夹内生成一个.git隐藏文件夹 ***千万不能删这个文件夹***

[![](/images/posts/git-guide/git-hidden-folder.png)](/images/posts/git-guide/git-hidden-folder.png)

#### 2.查看状态

```bash
git status
```
检查哪些文件被修改、哪些需要添加至版本管理

#### 3.添加文件到暂存区

```bash
git add 文件名      # 添加指定文件
git add .          # 添加所有文件
```
使用`git add .`前，建议配置`.gitignore`文件过滤依赖包、编译文件等

#### 4.提交到本地仓库
```bash
git commit -m "feat: 添加用户登录功能"
```
```
type(必须): 功能类别
  - feat：新功能
  - fix：修复bug
  - test：增加测试
  - revert：回滚版本

subject(必须): 简短描述，不超过50字符，中文更佳
```

#### 5.远程同步
##### 1.在GitHub上新建一个仓库

[![](/images/posts/git-guide/github-new-repo.png)](/images/posts/git-guide/github-new-repo.png)

##### 2.连接远程仓库
```bash
git remote add origin https://github.com/用户名/项目名.git
git remote add origin
```
##### 3.推送代码
```bash
git push -u origin main
```

[![](/images/posts/git-guide/git-push.jpg)](/images/posts/git-guide/git-push.jpg)

##### 4.拉取代码
```bash
git pull origin main
```
>push 前最好先 pull，提前解决协作冲突

###### 5.分支管理
```bash
# 创建并切换分支
git checkout -b feature-login

# 查看分支
git branch

# 合并分支
git merge feature-login
```
>合并前应在 GitHub/Gitee 创建 Pull Request(PR)，邀请审查后再合并

#### 6.避坑指南
##### 1. 版本回退
```bash
# 安全回退（修改保留在暂存区）
git reset --soft HEAD~1

# 彻底回退（丢弃未提交修改）
git reset --hard HEAD~1
```
##### 2.查看提交历史
```bash
git log
```
## 总结

多敲多练，熟能生巧！ 掌握以上内容，就可以进行简单的团队协作了

---

