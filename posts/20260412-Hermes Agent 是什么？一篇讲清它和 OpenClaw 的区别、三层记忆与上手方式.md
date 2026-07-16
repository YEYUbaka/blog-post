---
title: "Hermes Agent 是什么？一篇讲清它和 OpenClaw 的区别、三层记忆与上手方式"
date: 2026-04-12T12:00:00+08:00
categories: "学习笔记"
id: "20260412"
cover: "/images/posts/Hermes Agent 是什么？一篇讲清它和 OpenClaw 的区别、三层记忆与上手方式/cover.webp"
tags:
  - AI
  - Agent
  - Hermes
  - OpenClaw
  - 2026
recommend: false
top: false
hide: false
author: "YEYUbaka"
description: 如果你最近一直在看 OpenClaw，那大概率也刷到过 Hermes Agent。这篇文章不讲营销话术，只用项目资料和官方文档，把它的定位、自动长 Skill、三层记忆、工具机制和快速上手方式一次讲清。
showToc: true
---

:::note{type="success"}
如果最近 AI Agent 圈你只记住了一个关键词，那多半是 OpenClaw；但如果你再往下看一层，很快就会碰到 Hermes Agent。它吸引人的地方不是“又多了一个终端壳子”，而是它把学习、记忆和工具组织能力一起塞进了系统里。本文按项目资料和官方文档重新整理，重点讲清三个问题：它到底是什么、它和 OpenClaw 差在哪、普通用户该怎么上手。
:::

## Hermes Agent 是什么

先说结论：**Hermes Agent 不是单纯的聊天终端，而是一套“会随着使用逐渐变熟”的 Agent 系统。**

这个项目来自 Nous Research。官方在 README 里给它的定位很直接，核心关键词就是 `self-improving`。翻成更好理解的话，它想做的不是一次性助手，而是一个会在长期使用里不断积累经验的 Agent。

它有几个比较明显的特征：

- 不绑模型厂商，你可以接 OpenRouter、OpenAI、Anthropic、GitHub Copilot，或者任意兼容 OpenAI API 的自定义端点
- 不绑运行位置，既能在本地 CLI 里对话，也能挂到 Telegram、Discord、Slack、WhatsApp、Signal、Email 这类入口
- 不只是“会调工具”，而是会把做过的事沉淀成可复用的 Skills 和记忆
- 能跑在本地，也能跑在 VPS、Docker、SSH、Modal 这类环境里，不一定非得跟着你的笔记本走

![Hermes 欢迎界面](/images/posts/hermes-agent/hermes-welcome.jpg)

*▲ 启动后的 Hermes 更像一个长期在线的 Agent 工作台，而不只是一次性对话框*

---

## 它和 OpenClaw 到底差在哪

很多人会把 Hermes 当成 OpenClaw 的“平替”，这话不能说全错，但也不够准确。

它们解决的是相近问题，路线却不一样。

OpenClaw 更像一个强调控制力和可审计性的框架。你可以很明确地决定它有哪些 Skill、怎么工作、能碰哪些能力。Hermes 则更像是在这个基础上，再往前加了一层“自我积累”：它不只执行任务，还会在执行后把方法留下来。

你可以先记这张表：

| 对比项 | OpenClaw | Hermes Agent |
| :--- | :--- | :--- |
| 主要感觉 | 你来调教 Agent | Agent 会边用边长经验 |
| Skill 来源 | 手动写、手动装、手动维护 | 系统自动沉淀，也能继续装社区 Skill |
| 长期记忆 | 更依赖你主动维护 | 内建持久记忆和会话检索 |
| 默认运行姿态 | 更像按需启动 | 更适合长期在线或后台运行 |
| 适合谁 | 喜欢强控制、强透明 | 想要 Agent 越用越顺手 |

所以我更愿意把 Hermes 理解成 OpenClaw 旁边的另一条路线，而不是谁简单替代谁。

---

## 最核心的能力：它会自己长 Skill

Hermes 最值钱的地方，不是工具数量，而是它把“做完一件事之后要不要沉淀方法”这件事产品化了。

官方文档里把这一层叫做 **procedural memory**，也就是“程序化经验”或者“做事方法的记忆”。它不是只把信息记下来，而是把一套能复用的流程记下来。

从使用效果上看，它的闭环可以理解成四步：

1. 先把任务跑完
2. 在过程中识别哪些步骤是稳定、可复用的
3. 自动创建或更新 Skill
4. 下次遇到类似问题时优先调用这个 Skill

官方文档还写得更细：Hermes 往往会在这些时机生成 Skill：

- 完成了一个复杂任务，尤其是工具调用比较多的时候
- 中间踩了坑，但最后找到了可行路径
- 用户纠正了它的做法
- 它发现了一条值得固定下来的非平凡工作流

![Hermes Skills / Memory 相关工作流示意](/images/posts/hermes-agent/hermes-skill-system.jpg)

*▲ 它更像是在把一次任务里的“做法”整理成下次还能继续用的能力，而不是只留下结果*

这也是为什么很多人会觉得：OpenClaw 更像“你亲手养出来的龙虾”，Hermes 更像“会自己长本事的龙虾”。

---

## 三层记忆为什么重要

官方文档把 Hermes 的持久记忆拆成 `MEMORY.md` 和 `USER.md` 两部分，前者偏环境、项目和经验，后者偏你的偏好、沟通风格和习惯。另外它还支持 `session_search`，会把历史会话存进 SQLite，并用 FTS5 做全文检索。

如果从实际体验去理解，我觉得可以把它看成三层：

| 记忆层 | 主要放什么 | 作用 |
| :--- | :--- | :--- |
| 当前会话上下文 | 这轮任务正在做什么 | 保证它不会在当前对话里掉线 |
| 持久记忆 | 你的偏好、项目事实、长期约定 | 让它跨会话还能记住“你是谁、这是什么项目” |
| Skill 记忆 | 从任务里沉淀出来的方法 | 让它下次不必从零再试一遍 |

这里要注意一点：这个“三层”是我根据官方文档和实际工作方式做的理解性拆分，不是 Hermes 文档里的固定术语，但它很适合拿来理解 Hermes 的优势。

再直白一点说：

- 普通聊天机器人常见的问题是“这轮会，下一轮忘”
- Hermes 试图解决的是“这轮会，下一轮还记得，而且还会更熟”

![Hermes 模块拆解记录](/images/posts/hermes-agent/hermes-research-summary.jpg)

*▲ 把项目结构拆开后会发现，Skills、Memory、Gateway、Toolsets 本来就是 Hermes 设计里的核心部件*

---

## 为什么工具多不是重点，按需启用才是重点

很多人第一次看 Hermes，会先被“40+ 内置工具”吸引住。

但真正更重要的，其实是它的 **toolsets** 机制。

Hermes 不是简单把所有工具全开给 Agent，而是把工具按场景分组，按平台、按任务需要启用。官方文档里能看到它把 Web、Terminal、File、Browser、Vision、Image Gen、TTS、Skills、Memory 这些能力都拆成了可控的工具集。

这样做有三个直接好处：

- 权限更收敛，不会让本来只该查资料的 Agent 顺手拿到不必要的执行权限
- 上下文更干净，不必每轮都把无关工具塞进提示词
- 响应更稳，工具越克制，跑偏和误调用的概率越低

所以 Hermes 的重点从来不是“工具越多越厉害”，而是“该给什么工具时给什么工具”。

---

## 为什么很多人把它看成 OpenClaw 的另一条路线

把前面的点串起来，就比较容易理解 Hermes 为啥会被频繁拿来和 OpenClaw 放在一起讲。

两者的共同目标，都是让 AI 从“会聊天”走向“能做事”。区别在于：

- OpenClaw 更强调把规则和能力清楚交到你手里
- Hermes 更强调系统自己累积经验、自己复用经验

所以如果你特别看重这些点：

- 我想完全知道 Agent 为什么这么做
- 我愿意手动维护 Skill 和行为规范
- 我希望控制粒度尽可能细

那 OpenClaw 这条路会更顺手。

如果你更在意这些点：

- 我不想每次都从零教一遍
- 我希望它做得越多越像我的长期助手
- 我想把它放到后台长期跑

那 Hermes 的吸引力就会更大。

---

## 为什么安装时很多教程会先让你填 OpenRouter API Key

这个问题很常见，根源其实只有一句话：**Hermes 自己不是模型，它是 Agent 的系统层。**

它负责的是：

- 记忆
- 工具
- Skills
- 网关入口
- 学习循环

真正输出 Token、做推理、给回答的，还是你背后接的模型服务。所以官方 Quickstart 里第一步安装完之后，很快就会进入 provider 和 model 的选择。

很多教程喜欢先拿 OpenRouter 起步，原因也不复杂：

- 一个 Key 能接很多模型，试错成本低
- 切换模型方便，适合比较不同效果
- 对新手来说，起步路径比较短

但这不等于 Hermes 只能用 OpenRouter。按照官方文档，OpenAI、Anthropic、GitHub Copilot、Kimi、DeepSeek、Qwen，以及任意兼容 OpenAI API 的自定义端点都可以接。

还有一个很容易忽略的点：**Hermes 要求模型至少有 64K 上下文窗口**。这个限制是官方 Quickstart 明写的，因为多步工具调用、记忆和长上下文任务都需要足够大的上下文空间。

:::note{type="warning"}
截至 2026 年 4 月 12 日我查官方 Quickstart 时，Hermes 对 Windows 的推荐路径是：先安装 WSL2，再在 WSL2 里运行 Linux 安装命令。也就是说，如果你看到旧教程里还在直接演示原生 Windows 一键安装，优先以官方文档为准。
:::

---

## 5 分钟快速上手

如果你只是想先把它跑起来，步骤其实不复杂。

### 1. 安装

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 安装后重新加载 shell
source ~/.bashrc
```

### 2. 选 provider、模型和工具

```bash
hermes model
hermes tools
hermes setup
```

`hermes model` 用来选模型和服务商，`hermes tools` 用来决定启用哪些工具集，`hermes setup` 则适合重新走一遍完整配置。

![Hermes Quick setup](/images/posts/hermes-agent/hermes-quick-setup.jpg)

*▲ 如果只是先体验，走 Quick setup 就够了，后续再慢慢细配也不迟*

### 3. 开始对话

```bash
hermes
```

跑起来之后，你会看到欢迎界面、当前模型、可用工具和 Skills。

如果你准备把它接到消息平台，可以继续配置消息网关。官方 README 里常见的几个命令如下：

| 命令 | 用途 |
| :--- | :--- |
| `hermes` | 启动交互式 CLI |
| `hermes model` | 选择 provider 和模型 |
| `hermes tools` | 配置可用工具集 |
| `hermes setup` | 重新跑完整配置向导 |
| `hermes gateway` | 启动消息网关相关能力 |
| `hermes claw migrate` | 从 OpenClaw 迁移 |
| `hermes update` | 更新到最新版本 |
| `hermes doctor` | 做环境诊断 |

![Telegram 中与 Hermes 对话](/images/posts/hermes-agent/hermes-telegram-chat.jpg)

*▲ 真正接到消息入口之后，它的感觉就更像一个常驻在线的数字助理了*

---

## 它适合哪些场景

Hermes 不一定适合所有人，但有几类事情确实和它很搭。

### 1. 长期知识助手

如果你经常在同一个项目、同一套工作流里反复切换任务，那它的长期记忆和 Skills 会越来越有价值。你不需要每次都重新讲一遍背景。

### 2. 7×24 后台任务

官方有内置定时调度和消息网关，这让它很适合做日报汇总、通知跟踪、定时巡检、夜间任务这类持续型工作。

### 3. 持续内容创作

如果你经常写技术文章、做资料整理、维护自己的表达风格，那 Hermes 比“一次性问答型工具”更容易越用越顺。

### 4. 多平台 Bot

CLI、Telegram、Discord、Slack、WhatsApp、Signal、Email 这些入口打通后，它就不只是本地终端助手，而是一个真正可以长期在线的 Agent。

---

## 总结

Hermes Agent 最值得注意的，不是“它又接了多少平台”，也不是“它又加了多少工具”，而是它把学习和积累这件事做进了系统本身。

如果你更喜欢强控制、强透明、亲手把 Agent 一点点调出来，那 OpenClaw 依然是很强的一条路线。

但如果你已经开始觉得：

- 手动维护 Skill 很花时间
- 每次重新教 Agent 很累
- 你想要的是一个越用越像长期同事的系统

那 Hermes 的确值得认真试一下。

你可以把它和 OpenClaw 的区别，先记成一句最短的话：

**OpenClaw 更像“你来教它怎么做”，Hermes 更像“它会把做过的事慢慢学会”。**

---

**参考资料**:

- [Hermes Agent GitHub 仓库](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent 官方 Quickstart](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart/)
- [Hermes Agent Skills System 文档](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills)
- [Hermes Agent Persistent Memory 文档](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)
- [参考原文：很多人突然不玩小龙虾而用 Hermes Agent 了](https://mp.weixin.qq.com/s/fB4dLHu3BPHE4zWpuqoL4Q)
