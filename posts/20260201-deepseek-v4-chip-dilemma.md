---
title: "DeepSeek V4 发布：中国 AI 芯片困局下的技术突围"
date: 2026-02-01T23:00:00+08:00
categories: "技术分享"
id: "20260201"
cover: "/images/posts/deepseek-v4-chip-dilemma/cover.webp"
tags:
  - AI
  - 科技
  - DeepSeek
  - 芯片
  - 开源
  - 2026
description: "2026年2月中旬，DeepSeek发布V4模型，在编程能力上逼近GPT-5.3-Codex。更引人注目的是芯片禁令下的技术突围路径：如何在算力受限的情况下保持竞争力。"
hide: false
top: false
recommend: false
author: "YEYUbaka"
showToc: true
---


<!-- AI_GEN: 美国AI芯片出口管制演进时间线及对DeepSeek训练的影响 -->
::vhMusic{id="1456446090"}

:::note{type="import"}
2026年2月17日（中国农历新年后第三天），DeepSeek 发布了 V4 模型。与 R1 的"低成本推理"和 V3.1 的"混合推理"不同，V4 的定位非常明确——**编程能力**。在 HumanEval+ 和 MBPP 基准上，V4 的得分逼近了 GPT-5.3-Codex（2月6日发布），而 API 定价只有后者的约 15%。更值得关注的是 V4 的训练过程——据报道，团队曾尝试使用中国国产芯片进行部分训练，虽然最终因为技术问题导致发布延迟，但这个过程本身就意义非凡。
:::

## 从 V3.1 到 V4：聚焦编程的跳跃

DeepSeek V4 相对于 V3.1 的进步主要体现在三个维度：

**代码生成质量**：V4 在 HumanEval+（扩展版代码生成基准）上的 pass@1 从 V3.1 的约 78% 跃升到约 89%，仅比 GPT-5.3-Codex 的约 92% 少了 3 个百分点。考虑到价格差距（V4 的 API 定价大约是 Codex 的 1/7），这种性价比对于独立开发者和中小团队具有极强的吸引力。

**代码理解与重构**：V4 在跨文件理解和复杂重构任务上有了质的提升。能够理解一个项目中的多个文件之间的关系，并在不破坏现有功能的前提下做结构性修改。这是之前开源模型普遍做不好的一件事。

**工具使用（Tool Use）**：V4 原生支持更复杂的工具调用——它不只是"写代码"，还能"读写文件、执行 Shell 命令、搜索文档、运行测试"。这使 V4 成为了一个真正的编程 Agent，而不仅是代码补全工具。

## 芯片困局：被逼出来的创新

V4 发布背后有一个更值得关注的故事——芯片供应危机。

根据 Reuters 的深度报道，DeepSeek 原计划在 2025 年中发布 V4。但美国进一步收紧了对华 AI 芯片出口管制，导致 DeepSeek 原先依赖的 H800 GPU 供应变得更加紧张。团队做出了一个大胆的决定：**尝试用中国国产 AI 芯片（华为昇腾 910B）来完成部分训练**。

这个尝试最终导致了至少 3-4 个月的延迟。国产芯片的软件生态（驱动、编译器、深度学习框架适配）远不如 NVIDIA CUDA 成熟，大量时间花在了工程适配而非模型优化上。

但延迟不等于失败。据 Introl 的报道，V4 的最终训练方案是一个混合架构——大部分训练仍在 H800 上完成，但部分模块（尤其是中文语料的 Embedding 层）成功使用了国产芯片。这种"混合训练"的方案虽然效率不如纯 NVIDIA 方案，但证明了一条在芯片禁令下可行的技术路径。

更重要的是，这次经历迫使 DeepSeek 团队在国产芯片的软件栈上积累了宝贵的经验。一位接近 DeepSeek 的人士对媒体表示：「用国产芯片的过程很痛苦，但痛苦的经历会变成壁垒。那些只用 NVIDIA 的团队，一旦芯片被断供就抓瞎了。我们至少知道有哪些坑。」

## 对国内 AI 生态的示范效应


<!-- AI_GEN: 华为昇腾 vs NVIDIA GPU在AI训练场景的性能对比图 -->
DeepSeek V4 的"芯片困局突围"对国内 AI 生态有重要的示范意义。

在此之前，国内大模型团队对待芯片禁令的态度大致分为两种：一是"能买就买，囤货为主"（大部分创业公司），二是"国产芯片做不了大模型训练，等吧"（悲观派）。DeepSeek 证明了第三种可能：**用国产芯片做部分训练是可行的，虽然有代价，但这份代价在长期来看是值得的。**

华为昇腾团队和 DeepSeek 在 V4 开发过程中建立了深度合作关系。据报道，昇腾的软件团队根据 DeepSeek 的反馈在 6 个月内修复了超过 100 个关键 bug，并添加了多项针对大模型训练场景的优化。这种"模型团队驱动芯片工具链改进"的模式，可能比"芯片厂商自己闭门造车"更有效。

当然，华为昇腾 910B 在绝对性能上仍远不如 NVIDIA H100/B200。但 DeepSeek 的故事说明：工程创新可以在一定程度上弥补硬件差距。对于那些没有几百亿买 GPU 的团队来说，这是一个重要的启示。

## V4 的局限

客观地说，V4 与 GPT-5.3-Codex 之间仍有差距。主要体现在：

- **多语言编程**：V4 在 Python 上表现最好，但在 Rust、Go、TypeScript 上的表现与 Codex 有约 10-15% 的差距
- **代码安全性**：V4 生成代码中的安全漏洞检出率高于 Codex（即安全性更差）
- **推理速度**：在代码推理速度上 V4 比 Codex 慢约 30%（部分原因是硬件差异）

但这些差距对于 V4 的目标用户群体——独立开发者、中小团队、成本敏感型企业——来说，通常是可以接受的。毕竟，以 1/7 的价格获得 85-90% 的能力，这个性价比足够有说服力。

## 结语

DeepSeek V4 的故事不只是关于一个模型有多强。它是关于**在外部约束下如何保持创新动力**的故事。芯片禁令原本是美国遏制中国 AI 发展的手段，但某种程度上，它也倒逼了中国 AI 团队在工程优化和芯片适配上做出一些"如果不被逼就不会做"的创新。

到了2026年7月初，卡码大模型报道了另一个好消息：**DeepSeek V4 宣布降价 75%，V4-Pro 永久降价 75%。** 这对国内开发者来说是一个巨大的利好——V4 已经足够强了，现在又足够便宜了。用卡码大模型的话来说，2026年的大模型竞争已经从"有没有模型"进入到了"模型够不够好用、够不够便宜"的阶段。DeepSeek 用 "高能力 + 超低价格" 的组合拳，正在重新定义开源模型的市场竞争力。

V4 不会是最后一个在芯片困局中诞生的中国 AI 模型。接下来的关键看点是：DeepSeek 的下一代模型能否做到"完全用国产芯片训练"——如果能，那将是一个真正的转折点。

---
*参考来源：*
- [Introl — DeepSeek V4 Targets Coding Dominance with Mid-February Release](https://introl.com/blog/deepseek-v4-february-2026-coding-model-release)
- [卡码大模型 — DeepSeek V4 降价 75% 实测](https://notes.kamacoder.com/llm/news/deepseek-v4-price.html)
- [卡码大模型 — DeepSeek V4-Pro 永久降价 75%](https://notes.kamacoder.com/llm/news/deepseek-v4-pro-permanent-price-cut.html)
- [Reuters / Reddit — DeepSeek's next AI model delayed by attempt to use Chinese chips](https://www.reddit.com/r/LocalLLaMA/comments/1mpu8ot/deepseeks_next_ai_model_delayed_by_attempt_to_use/)
- [Decode the Future — DeepSeek R2 Release Date: Status & Rumors (2026)](https://decodethefuture.org/en/deepseek-r2-explained/)
