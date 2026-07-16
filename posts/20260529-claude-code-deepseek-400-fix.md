---
title: "最新版 Claude Code 接入 DeepSeek 报错 API Error 400？降级 2.1.153 一步解决"
date: 2026-05-29T12:00:00+08:00
categories: "技术分享"
id: "20260529"
cover: "/images/posts/claude-code-deepseek-400-fix/cover.webp"
tags:
  - Claude Code
  - DeepSeek
  - API
  - 踩坑记录
  - 2026
recommend: true
top: false
hide: false
author: "YEYUbaka"
description: "最新版 Claude Code 接入 DeepSeek 模型时报错 API Error 400，原因是 DeepSeek 的 /anthropic 端点不支持 messages 数组中的 system 角色。降级到 2.1.153 版本即可解决，本文记录完整排查和修复过程。"
showToc: true
---

::vhMusic{id="3361870496"}

:::note{type="warning"}
**如果你正在用最新版 Claude Code 对接 DeepSeek 的 Anthropic 兼容接口**，突然发现报 `API Error: 400` 死活连不上——不用折腾 API key 和网络，大概率是 Claude Code 新版本改了请求格式，跟 DeepSeek 不兼容了。

这篇文章帮你搞清楚报错的真正原因，并用最简单的降级操作恢复正常。
:::

## 问题现象

最近把 Claude Code 更新到最新版后，配置 DeepSeek 模型时报错，连最基本的对话都发不出去。完整报错信息如下：

```
API Error: 400 Failed to deserialize the JSON body into the target type:
messages[1].role: unknown variant `system`, expected `user` or `assistant`
at line 1 column 18790
```

核心信息就一句：**DeepSeek 不认识 `system` 这个 role，它只认 `user` 和 `assistant`**。

## 原因分析

### 问题出在 Anthropic API 兼容层的差异

Claude Code 发起 API 请求时，会把 system prompt（系统提示词）夹在 `messages` 数组里，给每条 message 标记 `role: "system"`。这在 Anthropic 官方 API 里是完全合法的——Anthropic 同时支持两种方式传递 system prompt：

- **顶层参数**：`system` 作为请求体的顶层字段（传统方式）
- **messages 内嵌**：`role: "system"` 放在 `messages` 数组里（新版方式）

但 DeepSeek 的 `/anthropic` 兼容端点是按传统方式实现的——**它只支持顶层 `system` 参数，不支持 `messages` 数组里出现 `system` 角色**。当 Claude Code 新版把 system prompt 塞进 messages 数组时，DeepSeek 服务端反序列化 JSON 直接失败：

```
unknown variant `system`, expected `user` or `assistant`
```

说白了就是：DeepSeek 的 messages 数组预期里只有 `user` 和 `assistant` 两种角色，冒出一个 `system`，它不知道该拿它怎么办。

### 哪个版本开始出问题的？

Claude Code 在某个版本之后调整了 system prompt 的发送方式，从顶层参数改成了 messages 内嵌。具体从哪个小版本开始的我没逐个测，但可以确定：

- **2.1.153**：还在用传统的顶层 `system` 参数 → 兼容 DeepSeek
- **2.1.154+**（含当前最新版）：改用 messages 内嵌 `role: "system"` → 不兼容 DeepSeek

所以解决方案很直接——**降级到 2.1.153**，这是最后一个兼容 DeepSeek 的版本。

## 解决方案

### 方案一：CLI 降级（PowerShell）

如果你用的是命令行版 Claude Code，一条命令搞定：

```powershell
npm install -g @anthropic-ai/claude-code@2.1.153 --force
```

`--force` 是为了覆盖已安装的最新版，避免 npm 因为版本"降级"而拒绝安装。

安装完成后验证版本：

```bash
claude -v
```

输出 `2.1.153` 就说明降级成功了。

![PowerShell 降级流程](/images/posts/claude-code-deepseek-400-fix/powershell-downgrade.png)

*▲ PowerShell 中执行 npm install 降级 Claude Code 到 2.1.153 的完整流程*

### 方案二：VS Code 插件降级

如果你用的是 VS Code 里的 Claude Code 插件，操作也很简单：

**第一步：关掉自动更新**

打开 VS Code 扩展面板（`Ctrl+Shift+X`），搜索 "Claude Code for VS Code"，点击插件齿轮 ⚙️ → **关闭自动更新**。否则你一降级它又自动升回去了。

**第二步：安装指定版本**

同样在插件齿轮菜单里，选择 **"安装另一个版本..."（Install Another Version...）**，在弹出的版本列表里选 **2.1.153**。

![VS Code 插件降级操作](/images/posts/claude-code-deepseek-400-fix/vscode-plugin-downgrade.png)

*▲ 在 VS Code 插件商店中关闭自动更新，并通过"安装另一个版本"选择 2.1.153*

等安装完成，重新加载 VS Code 窗口，API Error 400 就不会再出现了。

## 关于版本 2.1.153

为什么选 2.1.153 而不是别的版本？这里有三个考量：

1. **API 格式**：2.1.153 是最后一个用传统顶层 `system` 参数发请求的版本。从 2.1.154 开始，system prompt 的发送方式变了，messages 数组里出现了 `role: "system"`，直接触发 DeepSeek 的 400 错误。

2. **功能完整**：2.1.153 功能上跟当前最新版差别不大，sub-agent、skills、MCP、hook 等核心能力都有，日常使用完全够。

3. **稳定性**：之前那波 Windows 兼容性问题（claude.exe 变 1KB）我在 2.1.112 上踩过坑，2.1.153 是在那之后的版本，Windows 下运行稳定，不需要额外折腾。

总结成一个简单的判断：

:::note{type="info"}
- 接入 **Anthropic 官方 API** → 随便升最新版，不受影响
- 接入 **DeepSeek / 第三方 Anthropic 兼容接口** → 锁定 2.1.153，别手贱更新
:::

## 总结

这个问题说到底是个**API 兼容性错位**：

- Anthropic 在自己生态里改了 system prompt 的传递方式（顶层 → messages 内嵌）
- DeepSeek 的兼容层只实现了传统的顶层 `system` 参数
- Claude Code 新版跟着 Anthropic 走，第三方适配没跟上

社区里其实有不少人遇到了同样的问题，CSDN 和博客园上都有讨论。解决方式都一样：**降级到 2.1.153，关闭自动更新**。

两分钟的操作，值得一记。

---

**参考资料**：

- [ClaudeCode接入DeepSeek报错400 - CSDN](https://blog.csdn.net/bc202205/article/details/161505390)
- [Claudecode降级2.1.153解决API Error: 400 - 博客园](https://www.cnblogs.com/xzaxs/p/20212017#!comments)
