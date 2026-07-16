---
title: "Claude Code 2.1.36+ 的 billing header 在破坏你的 prompt cache？一行命令修复"
date: 2026-05-25T12:00:00+08:00
categories: "技术分享"
id: "20260525"
cover: "/images/posts/缓存命中修复.png"
tags:
  - Claude Code
  - 踩坑记录
  - API
  - 代理
  - Prompt Cache
  - 2026
recommend: true
top: false
hide: false
author: "YEYUbaka"
description: "升级 Claude Code 2.1.36 后，第三方 API 代理的 prompt caching 突然失效？原因是新版在 system prompt 中插入了每次变化的 x-anthropic-billing-header。本文拆解根因、源码追踪、给出修复方案，并整理了各大代理的适配现状。"
showToc: true
---

:::note{type="warning"}
**如果你正在用第三方 API 代理玩 Claude Code**，并且最近发现 token 消耗暴涨、推理变慢、缓存命中率突然归零——那么你大概率被这个东西坑了。

这篇文章会帮你搞清楚：发生了什么、为什么、怎么修。文末还整理了 DeepSeek、智谱、vLLM 等主流代理的适配现状，帮你判断自己是否需要动手。
:::

## 一、问题现象

事情要从 Claude Code 的 **2.1.36 版本**说起。

2026 年 5 月，B站 UP 主**张司机在路上**发了一个视频（BV1m2LG6WEdH，当前播放量 7.6 万），标题直截了当——「如何修复 Claude Code 给第三方大模型用户挖的坑」。

视频里他用 `cc-tap`（一个 Claude Code 抓包工具）截取了一组对比数据：

同一个 session 里，给 Claude Code 连续发三轮简单的消息（Hello → Fine → Thank you）。正常情况下，第二个和第三个请求应该命中 prompt cache，大幅度减少 token 消耗。

但实际抓包结果是——**三轮对话的 system prompt 第一块，每次都不一样：**

```
第一轮: x-anthropic-billing-header: cc_version=...; cc_entrypoint=...; cch=97bd6
第二轮: x-anthropic-billing-header: cc_version=...; cc_entrypoint=...; cch=24c2d
第三轮: x-anthropic-billing-header: cc_version=...; cc_entrypoint=...; cch=ead88
```

注意看 `cch` 这个字段——5 位 hex 字符，每次都在变。

而 prompt cache 的原理是前缀哈希：从 system prompt 开头算起，任何一字节变了，整个缓存 key 就对不上了。所以第一轮刚建好的缓存，第二轮的 `cch` 一换，cache 直接 miss。第三轮再换，再 miss。

**结果就是：用第三方代理的用户，每轮对话都等于第一次对话——全量 token 消耗，没有任何缓存命中。**

---

## 二、问题出在哪？

我们先复习一下 Prompt Cache 的工作方式（张司机的前几期视频有详细讲解）：

1. 服务端按 **Tools → System → Messages** 的顺序拼接前缀
2. 请求里每个带 `cache_control` 标记的 block 都定义了一个 **breakpoint**（缓存检查点）
3. 每个 breakpoint 的缓存 key = 从开头到该 block 末尾所有内容的哈希值

Claude Code 的请求通常有三个 breakpoint：
- 第一个：System 第二个 block 末尾
- 第二个：System 第三个 block 末尾
- 第三个：User message 上

**而关键问题是：`x-anthropic-billing-header` 被塞在了 System 的第一个 block 里——排在所有 breakpoint 的最前面。**

所以这条 header 一变，后面所有 breakpoint 的哈希全部跟着变。三个缓存检查点全部 miss。

:::note{type="info"}
**为什么 Anthropic 官方服务不受影响？**

因为 Anthropic 自己的服务端知道这段内容是自己塞进去的，算缓存 key 的时候会**跳过这个 block**。但第三方大模型转发服务不知道这个细节，老老实实把整段 system prompt 拿来算哈希——所以每次都算出一个新 key，缓存永远对不上。
:::

这个问题其实从 2026 年 2 月就开始被社区报告了。GitHub Issues 里有大量相关讨论，但 Anthropic 一直没有正面回应。原因也很简单：**他们自己的服务不受影响。**

---

## 三、源码追踪：cch 到底怎么生成的？

张司机在视频里直接扒了 Claude Code 的源码，追踪了整个链路。

### JS 层：`getAttributionHeaders`

在 `src/constant/system.ts` 中可以找到这个函数——它负责构造 billing header 字符串：

```typescript
// 伪代码还原
function getAttributionHeaders() {
  if (!isAttributionHeadersEnabled()) return "";

  let cch = "00000";  // 静态占位符
  if (featureFlag) {
    // cch 在 JS 层始终是 "00000"
  }

  return `x-anthropic-billing-header: cc_version=${version}; cc_entrypoint=${entrypoint}; cch=${cch}`;
}
```

注意看——`cch` 在 JS 层是一个**静态占位符 `00000`**。真正的值在哪计算的呢？

### Zig 原生层：`Attestation.zig`

源码中的注释大方地承认了真相：

> "真正的 cch 是由 NativeHttpGen 在请求发出的最后一刻计算的，会覆盖占位符。"

关键点：Claude Code 是用 **Bun**（一个用 Zig 写的 JS 运行时）编译打包成独立二进制的。Anthropic 不仅用了 Bun，还 fork 了一份自己的版本 `bun-anthropic`，在其中加入了一个文件叫 `Attestation.zig`。

它做的事情非常精巧：

1. JS 层生成 header，其中 `cch=00000`（5 个零）
2. 因为 `00000` 和真实值长度相同（都是 5 位 hex），Content-Length 不变
3. Zig 原生层在请求发出的最后瞬间，把 `00000` 直接原地替换成真实的 cch 值
4. Buffer 不需要重新分配，整个过程对 JS 层完全透明

**所以从外面看，cch 就像凭空变出来的一样——每次都不一样，你在 JS 里抓不到它的生成逻辑。**

### 那 cch 到底是干什么的？

一句话：**防白嫖。**

Claude Code 的登录走的是 OAuth，拿到 Token 后绑定你的 Pro 或 Max 订阅。订阅价格比直接 API 按量付费便宜很多。于是有人想：「我把 Token 从二进制里抠出来，自己写代码调用 API，不就能用订阅价跑任意请求了？」

那服务器怎么识别一个请求是真的来自 Claude Code 客户端、还是某个山寨程序伪造的？

答案就是 cch——它由 Zig 原生层计算，是一段只存在于 Bun 运行时内部的客户端签名。山寨程序根本拿不到真值，一发请求就露馅了。

不过嘛……这套防御机制也不是铁板一块。GitHub 上已经有了开源项目（比如 Sub2API）把整套 cch 算法完整复刻了。藏在 Zig 层的逻辑，在开源社区面前早就透明了。

---

## 四、怎么修复？

回到代码里，追一下 `isAttributionHeadersEnabled` 这个函数：

```typescript
function isAttributionHeadersEnabled() {
  // 第一行就检查环境变量
  if (!process.env.CLAUDE_CODE_ATTRIBUTION_HEADER) return false;
  // ...
}
```

所以解决方案就是一句话：

**设置环境变量 `CLAUDE_CODE_ATTRIBUTION_HEADER=0`，让整个 billing header 直接返回空串。**

### 具体操作

编辑 `~/.claude/settings.json`，在 `env` 段中加一行：

```json
{
  "env": {
    "CLAUDE_CODE_ATTRIBUTION_HEADER": "0"
  }
}
```

如果你用的是命令行启动，也可以在终端里直接设：

```bash
# Linux / macOS
export CLAUDE_CODE_ATTRIBUTION_HEADER=0

# Windows PowerShell
$env:CLAUDE_CODE_ATTRIBUTION_HEADER=0
```

设完之后**重启 Claude Code** 即刻生效。

### 怎么验证？

用 `cc-tap` 或任何抓包工具看请求内容。设置前，system prompt 的第一个 block 是：

```
block 1: x-anthropic-billing-header: cc_version=2.1.143.f09; cc_entrypoint=cli; cch=0f646;
block 2: You are Claude Code, Anthropic's official CLI for Claude.
block 3: ...
```

设置后，billing header 整个消失：

```
block 1: You are Claude Code, Anthropic's official CLI for Claude.
block 2: ...
```

System block 数量从 3 个变成 2 个。之后 `cch` 不再变化，prompt cache 恢复正常。

---

## 五、各大代理适配现状

不是所有第三方代理都需要你自己动手。以下是基于社区实测整理的各家适配情况：

### ✅ 无需手动修复

| 平台 | 状态 | 来源 |
|------|------|------|
| **DeepSeek API** | 已自行处理，缓存命中率 ~99% | 社区实测：加不加环境变量，缓存命中率几乎没区别 |
| **智谱 GLM-5.1** | 服务端已过滤 billing header | 社区实测：加不加环境变量，服务端返回的缓存读取数量相同 |
| **vLLM (v>0.17.1)** | PR #36829 已修复（2026-03-12 合并） | 在 `serving.py` 中检测 billing header block 并直接 `continue` 跳过 |

vLLM 的修复方式非常直接——在收到 Anthropic 格式请求时，检查 system prompt 中的每个 block：

```python
# vllm/entrypoints/anthropic/serving.py
for block in anthropic_request.system:
    if block.type == "text" and block.text:
        if block.text.startswith("x-anthropic-billing-header"):
            continue  # 直接跳过整个 billing header block
        system_prompt += block.text
```

这样真正的 system prompt 内容（从 `"You are Claude Code..."` 开始）完全不受影响，billing header 被原地丢弃。

### ⚠️ 建议手动设置

如果你用的是**自建代理、小众转发服务、LiteLLM、Ollama 本地部署**，或者不确定你的代理有没有做处理——**保险起见，直接加环境变量。**

Anthropic 官方文档也说得很清楚：

> "Claude Code 还会在系统提示前面添加一个简短的归属块，其中包含客户端版本和从对话派生的指纹。Anthropic API 在处理前会删除此块，因此不会影响第一方提示缓存。**如果您的网关实现了自己的提示缓存（以完整请求体为键），请设置 CLAUDE_CODE_ATTRIBUTION_HEADER=0 以省略它。**"

来源：[Claude Code 官方文档 — LLM Gateway](https://code.claude.com/docs/zh-CN/llm-gateway#llm-gateway)

---

## 六、社区评论精选

张司机视频下的评论区也有不少有价值的信息，摘几条：

> **评论1：** "我用的是 DeepSeek 的 Anthropic-compatible API，链路是 `Claude Code → 本地抓包转发代理 → DeepSeek`。未设置环境变量时，抓包确认 `cch` 确实在变。设置 `CLAUDE_CODE_ATTRIBUTION_HEADER=0` 后 billing header 消失。我也问了 DeepSeek 模型能否看到这个 header，它回答没看到。所以 DeepSeek 可能已经在适配层过滤了。"

> **评论2：** "测试了智谱的 Coding Plan GLM-5.1，加不加环境变量，服务端返回的缓存读取数量一样，说明智谱也对这个头做过处理了。"

> **评论3：** "早就修复了，现在才发视频。vLLM 在 PR #36829（3 月 12 日合并，v>0.17.1）里修了——检测到 billing header block 直接 skip。"

> **评论4：** "这个 cch 是 CC 客户端的签名，用于保护他的订阅服务，是上次源码泄露后被发现的。"

> **评论5：** "题外话：如果你好奇 cch 的完整生成算法，GitHub 上已经有开源项目复刻了整个逻辑。Anthropic 藏在 Zig 层的东西，社区早就扒干净了。"

---

## 七、总结

整个故事的脉络其实很清晰：

1. **Claude Code 2.1.36+** 在每个 API 请求的 system prompt 最前面插入了一行 `x-anthropic-billing-header`
2. 其中的 `cch` 字段是 5 位 hex，每次请求变化，由 Zig 原生层在请求发出前瞬间生成
3. 它的原始目的：防止第三方用订阅 Token 白嫖 API（客户端签名验证）
4. **副作用**：第三方代理在计算 prompt cache 哈希时包含了这段变化的内容 → 缓存 key 每次不同 → 缓存全部 miss
5. Anthropic 官方服务不受影响（它们会跳过这个 block 算缓存），所以一直没有积极处理
6. **修复**：加一行环境变量 `CLAUDE_CODE_ATTRIBUTION_HEADER=0`，重启 Claude Code

加一个环境变量，重启，你的 prompt cache 就回来了。

如果你在用第三方 API 代理，不妨顺手设上。哪怕你的代理声称已经适配了，多一层保险总是好的。

---

## 参考资源

- [B站视频：如何修复Claude Code给第三方大模型用户挖的坑 — 张司机在路上](https://www.bilibili.com/video/BV1m2LG6WEdH/)
- [Claude Code 官方文档 — LLM Gateway](https://code.claude.com/docs/zh-CN/llm-gateway#llm-gateway)
- [Claude Code 官方文档 — 环境变量参考](https://code.claude.com/docs/zh-CN/settings)
- [vLLM PR #36829: fix Anthropic billing header breaking prompt cache](https://github.com/vllm-project/vllm/pull/36829)
- [Claude Code cch 生成机制相关分析](https://aiorang.com/article/P1mOhMs.html)
