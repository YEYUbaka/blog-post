---
title: "OpenClaw核心技能安装指南"
date: 2026-03-13T11:22:40+08:00
categories: "技术分享"
id: "20260313"
cover: /images/posts/openclaw1.png
tags:
  - OpenClaw
  - AI
  - 教程
  - 指南
  - 2026
recommend: true
top: false
hide: false
author: "YEYUbaka"
description: '导读：OpenClaw在2026年彻底爆火，但很多新手安装后却发现它只是个"会聊天的终端"。原因很简单：你没装对Skills。面对ClawHub上超过13,000个技能插件，盲目安装不仅卡顿，还可能遭遇安全风险。本文基于GitHub高星项目及社区实测数据，为你筛选出最适合新手的20个核心Skills，覆盖办公、开发、生活全场景，复制命令即可生效。'
showToc: true
---

:::note{type="success"}
本篇文章是一份针对OpenClaw新手的核心技能安装指南，旨在解决新手安装后功能单一、性能卡顿和安全风险的问题。它基于社区实测和高星项目，从超过13,000个技能插件中筛选出最适合新手的20个"真神"Skills，并提供了详细的一键安装命令
:::

**参考资料**:

- [别乱装！2026年OpenClaw新手必装的20个"真神"Skills](https://mp.weixin.qq.com/s/YzMkLud3ITYAcetOCZ-hIw)
- [OpenClaw 完全部署指南：从入门到安全加固](https://zhuanlan.zhihu.com/p/2004187601276540473)
- [OpenClaw爆火，符合所有人利益的一场狂欢?](https://www.bilibili.com/video/BV1srwxzMEdn/)

::vhMusic{id="3338033128"}

## 什么是OpenClaw？

如果你比较关注AI内容，那你一定最近被OpenClaw（龙虾🦞）刷屏了，甚至连央妈都在讨论，那到底什么是OpenClaw呢？简而言之是能够掌管你电脑的AI助手甚至是你的数字员工，而且你还能通过各种消息平台(WhatsApp、Telegram、Slack、Discord、Signal、iMessage、飞书、钉钉等)来直接指挥它。在GitHub上甚至只用了两个月就超越了React这一现代Web应用根基

>2026 年 3 月 3 日，OpenClaw 正式超越 React，成为 GitHub 上 Star 数最多的软件项目，总 Star 数突破 25 万。React——这个驱动了现代 Web 大部分应用的 JavaScript 框架——花了十多年才达到这个里程碑。而 OpenClaw 只用了大约 60 天。
>
>这些数字令人震撼：250,829 个 Star、48,274 个 Fork、1,075 位贡献者，以及 9,574 个开放 Issue。仅在过去两周内，仓库就新增了约 8 万个 Star。

:::picture
![央妈报道OpenClaw](/images/posts/openclaw-guide/cctv-discussion-1.png)
![OpenClaw GitHub数据](/images/posts/openclaw-guide/github-stars.jpg)
![央妈报道OpenClaw](/images/posts/openclaw-guide/cctv-discussion-2.png)
![OpenClaw安全问题](/images/posts/openclaw-guide/security-issue.png)
:::

我也在第一时间在Ubuntu虚拟机上实装使用了一段时间，现在推荐几个提升效率和体验的Skills。

---

## 前置准备：如何安装Skill？

在开始之前，请确保你已经完成了OpenClaw的基础部署。安装技能非常简单，只需在终端运行以下通用命令：

```bash
# 通用安装命令
npx clawhub@latest install <技能名称>

# 或者在OpenClaw对话中直接输入
/install <技能名称>
```

:::note{type="warning"}
**安全提示**：建议优先安装官方认证或GitHub高星（Star > 1k）的技能，避免使用来源不明的插件，以防API Key泄露。
:::

---

## 第一梯队：基础生存包（必装前5个）

>没有这些，OpenClaw只是一个普通的聊天机器人。

### 1. gog（Google Workspace全能管家）

- **官方链接**: [ClawHub/gog](https://clawhub.ai/steipete/gog)
- **安装命令**: `npx clawhub@latest install gog`
- **能干什么**:
  - 一句话管理Gmail、Google Calendar和Drive
- **实战**: "帮我给老板发封邮件，附上上周的会议纪要，并预约明天下午3点的会议。"
- **评价**: 安装量第一，配置最简单，职场人必备。

### 2. summarize（万物总结神器）

- **官方链接**: [ClawHub/summarize](https://clawhub.ai/steipete/summarize)
- **安装命令**: `npx clawhub@latest install summarize`
- **能干什么**:
  - 总结PDF、长网页、YouTube视频字幕、甚至整个代码库
- **实战**: "总结这个PDF文件的核心观点，生成300字摘要。"
- **评价**: 节省阅读时间的最强利器，准确率极高。

### 3. github（开发者第二大脑）

- **官方链接**: [ClawHub/github](clawhub.ai/steipete/github)
- **安装命令**: `npx clawhub@latest install github`
- **能干什么**:
  - 查Issue、提PR、管理仓库、自动Code Review
- **实战**: "查看我仓库 my-project 中未关闭的Bug列表，并创建一个修复分支。"
- **评价**: 程序员必备，让AI真正参与代码工作流。

### 4. file-manager（本地文件指挥官）

- **官方链接**: [ClawHub/file-manager](https://clawhub.ai/file-manager)
- **安装命令**: `npx clawhub@latest install file-manager`
- **能干什么**:
  - 读写、搜索、整理、重命名本地文件
- **实战**: "把下载文件夹里所有的PDF移动到'文档/2026报告'目录，并按日期重命名。"
- **评价**: 实现本地知识库交互的基础，注意配置好权限目录。

### 5. weather（零配置天气助手）

- **官方链接**: [ClawHub/weather](https://clawhub.ai/weather)
- **安装命令**: `npx clawhub@latest install weather`
- **能干什么**:
  - 无需API Key，快速查询全球天气，支持每日简报
- **实战**: "北京下周会下雨吗？如果会，提醒我周三带伞。"
- **评价**: 新手友好度满分，无需任何复杂配置即可使用。

---

## 第二梯队：办公与知识管理（效率倍增）

### 6. notion-connector

- **官方链接**: [ClawHub/notion](https://clawhub.ai/notion)
- **安装命令**: `npx clawhub@latest install notion`
- **能干什么**: 同步笔记、创建数据库条目、管理任务看板
- **实战**: "在Notion的'任务库'中新建一条任务：完成Q1财报分析，截止日期下周五。"

### 7. obsidian-sync

- **官方链接**: [ClawHub/obsidian](https://clawhub.ai/obsidian)
- **安装命令**: `npx clawhub@latest install obsidian`
- **能干什么**: 双向链接笔记管理，支持本地Markdown文件操作，隐私优先
- **实战**: "在我今天的日记里记录刚才的会议要点，并链接到'项目A'笔记。"

### 8. nano-pdf

- **官方链接**: [ClawHub/nano-pdf](https://clawhub.ai/nano-pdf)
- **安装命令**: `npx clawhub@latest install nano-pdf`
- **能干什么**: 轻量级PDF编辑、转换、提取文字、合并拆分
- **实战**: "把这个Word文档转成PDF，并提取第5页的表格数据。"

### 9. calendar-scheduler

- **官方链接**: [ClawHub/scheduler](https://clawhub.ai/scheduler)
- **安装命令**: `npx clawhub@latest install calendar-scheduler`
- **能干什么**: 跨平台日程冲突检测与自动预约（支持Outlook/本地日历）
- **实战**: "帮我找一个下周一到周三大家都空闲的1小时开会时间。"

### 10. rss-reader

- **官方链接**: [ClawHub/rss](https://clawhub.ai/rss)
- **安装命令**: `npx clawhub@latest install rss`
- **能干什么**: 订阅博客、新闻源，定时推送摘要
- **实战**: "订阅Hacker News和少数派，每天早上8点推送最新热点摘要。"

---

## 第三梯队：开发与极客工具（进阶必备）

### 11. code-executor

- **官方链接**: [ClawHub/code-executor](https://clawhub.ai/code-executor)
- **安装命令**: `npx clawhub@latest install code-executor`
- **能干什么**: 在沙箱环境中运行Python/Node.js代码，处理数据计算、绘图
- **实战**: "用Python画一张过去五年比特币价格走势的折线图。"

### 12. docker-controller

- **官方链接**: [ClawHub/docker](https://clawhub.ai/docker)
- **安装命令**: `npx clawhub@latest install docker`
- **能干什么**: 管理本地Docker容器（启停、日志查看、镜像构建）
- **实战**: "重启名为'web-server'的容器，并显示最近100行日志。"

### 13. ssh-connector

- **官方链接**: [ClawHub/ssh](https://clawhub.ai/ssh)
- **安装命令**: `npx clawhub@latest install ssh`
- **能干什么**: 安全连接远程Linux服务器，进行远程巡检和部署
- **实战**: "连接到生产服务器，检查CPU使用率，如果超过80%就重启服务。"
- **注意**: 务必配置SSH密钥白名单，限制可执行的命令。

### 14. api-tester

- **官方链接**: [ClawHub/api-tester](https://clawhub.ai/api-tester)
- **安装命令**: `npx clawhub@latest install api-tester`
- **能干什么**: 发送HTTP请求，测试API接口连通性，替代Postman
- **实战**: "向 https://api.example.com/users 发送GET请求，返回状态码和响应体。"

### 15. sql-explorer

- **官方链接**: [ClawHub/sql](https://clawhub.ai/sql)
- **安装命令**: `npx clawhub@latest install sql`
- **能干什么**: 连接数据库执行查询、生成报表（建议只读权限）
- **实战**: "查询上个月销售额最高的前10个产品，并生成CSV文件。"

---

## 第四梯队：生活与媒体创作（让AI更有趣）

### 16. media-downloader

- **官方链接**: [ClawHub/media-downloader](https://clawhub.ai/media-downloader)
- **安装命令**: `npx clawhub@latest install media-downloader`
- **能干什么**: 下载YouTube视频、B站视频、Spotify歌单（注意版权）
- **实战**: "下载这个B站视频的音频部分，保存到'音乐'文件夹。"

### 17. image-gen

- **官方链接**: [ClawHub/image-gen](https://clawhub.ai/image-gen)
- **安装命令**: `npx clawhub@latest install image-gen`
- **能干什么**: 调用Stable Diffusion或DALL-E 3生成图片
- **实战**: "生成一张'2077年赛博朋克风格的上海'图片，比例16:9。"

### 18. expense-tracker

- **官方链接**: [ClawHub/expense](https://clawhub.ai/expense)
- **安装命令**: `npx clawhub@latest install expense-tracker`
- **能干什么**: 记录日常开销，生成消费图表，数据存在本地
- **实战**: "记录今天午餐花费35元，类别'餐饮'，并显示本月餐饮总支出。"

### 19. news-digest

- **官方链接**: [ClawHub/news](https://clawhub.ai/news)
- **安装命令**: `npx clawhub@latest install news-digest`
- **能干什么**: 每日定时推送定制领域的新闻摘要（科技、财经等）
- **实战**: "每天早上9点推送关于'人工智能'领域的最新三条新闻。"

### 20. translate-pro

- **官方链接**: [ClawHub/translate](https://clawhub.ai/translate)
- **安装命令**: `npx clawhub@latest install translate-pro`
- **能干什么**: 多语言互译，支持文档、网页、对话实时翻译
- **实战**: "把这份英文合同翻译成中文，并保持原有的格式。"

---

## 新手避坑与建议

### 1. 不要一次性全装

建议先装**第一梯队**的5个，稳定运行一周后，根据实际需求按需安装。一次性安装过多会导致上下文过长，响应变慢，甚至引发逻辑冲突。

### 2. 权限最小化原则

涉及文件读写（`file-manager`）和网络操作（`ssh-connector`）的技能，务必在 `openclaw.json` 配置文件中限制其访问目录和白名单IP。

```json
{
  "skills": {
    "file-manager": {
      "allowedPaths": ["/home/user/documents"],
      "denyPaths": ["/etc", "/root"]
    },
    "ssh-connector": {
      "allowedHosts": ["192.168.1.100", "myserver.com"]
    }
  }
}
```

### 3. 关注Token消耗

定期查看日志，观察哪些技能最耗Token。像 `search` 类技能，建议在提示词中限制搜索次数（如"最多搜索3次"）。

### 4. 善用 find-skills

如果你找不到特定功能的技能，可以安装 `find-skills` 插件，直接问OpenClaw："我想做一个XXX功能，有什么推荐的技能吗？"它会帮你检索ClawHub。

---

## 总结

OpenClaw的强大不在于模型本身，而在于这些**Skills赋予它的"手脚"**。这20个技能是经过社区数万次验证的"黄金组合"，能帮助你从零搭建一个真正能干活的数字员工。

我的建议是：先从**第一梯队的5个基础技能**开始，熟悉OpenClaw的工作方式后，再根据自己的实际需求逐步添加其他技能。不要贪多，够用就好。

如果你已经安装了一些好用的OpenClaw Skills，或者有自己开发的独家技能，欢迎在评论区分享你的配置清单！

---

**相关链接**:
- [OpenClaw 官方文档](https://docs.openclaw.ai/)
- [ClawHub 技能市场](https://clawhub.ai/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
