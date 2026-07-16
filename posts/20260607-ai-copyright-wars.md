---
title: "AI 训练数据的版权之战：NYT 诉 OpenAI 与全球立法进展"
date: 2026-06-07T01:00:00+08:00
categories: "技术分享"
id: "20260607"
cover: "/images/posts/ai-copyright-wars/cover.webp"
tags:
  - AI
  - 科技
  - 版权
  - 训练数据
  - EU AI Act
  - 法律
  - 2026
description: "纽约时报诉OpenAI案进入关键阶段，EU AI Act的高风险管理规则即将于8月全面生效。AI训练数据的版权问题正在从法律灰色地带走向清晰——而这一过程将重塑整个AI产业。"
hide: false
top: false
recommend: false
author: "YEYUbaka"
showToc: true
---


<!-- AI_GEN: NYT诉OpenAI案的核心法律争议点思维导图 -->
::vhMusic{id="1447554651"}

:::note{type="import"}
2025-2026年，AI 训练数据的版权问题进入了法律战场。"纽约时报诉 OpenAI"案成为标志性案件，涉及 ChatGPT 是否在训练中"未经授权复制"了 NYT 的新闻内容。与此同时，EU AI Act 的高风险 AI 系统规则将于 2026年8月2日全面生效，要求训练数据透明化。这些法律进展正在重塑 AI 行业的基本规则——而"爬虫自由"的时代正在走向终结。
:::

## NYT 诉 OpenAI：AI 版权的地标案件

纽约时报（NYT）在 2023 年底起诉 OpenAI 和 Microsoft，指控 ChatGPT 的训练数据中包含了 NYT 的版权文章，且在生成回答时有时会逐字或接近逐字地复述 NYT 的内容。这是"传统版权方 vs AI 公司"系列案件中最引人注目的一桩。

到 2026年中，案件经过了两年的证据开示和听证，进入了实质性审理阶段。几个关键的法律争议点：

**"合理使用"（Fair Use）的边界**：OpenAI 辩称其训练过程属于"合理使用"——用于训练 AI 模型属于对版权材料的"转换性使用"。但 NYT 反驳说，ChatGPT 有时能生成几近逐字复述的新闻内容，这不是"转换性使用"，而是"未经授权的复制和分发"。

**"记忆"（Memorization）问题**：AI 模型在训练过程中"记住"（而非"学习"）部分训练数据已经被多项研究证实。如果 ChatGPT "记住"了 NYT 的整篇文章并能够复述，这在法律上是否构成"未经授权的复制"？

**训练数据透明度**：NYT 要求 OpenAI 披露其训练数据的完整来源列表。OpenAI 以"商业机密"为由拒绝。这一争议将成为案件结果的重要变量。

法律专家普遍认为，NYT 案的结果将为 AI 行业设定一个重要的先例——无论谁赢，都会影响未来 AI 公司获取和使用训练数据的方式。

## EU AI Act：8月2日的合规 deadline

EU AI Act 是全球最全面的 AI 法律框架。它在 2024年8月正式生效，采取了分阶段实施的策略。最关键的节点是 2026年8月2日——**高风险 AI 系统的合规要求将在这一天全面生效**。

对于 AI 训练数据而言，EU AI Act 的核心要求是：

1. **训练数据透明度**：高风险 AI 系统的开发者必须提供训练数据的摘要（包括数据来源、数据处理方式和潜在偏见评估）。虽然不需要公开完整的训练数据，但监管机构有权要求查看。

2. **版权合规**：AI 公司必须遵守欧盟版权法，包括发布训练数据中所包含的版权作品的摘要。这意味着 AI 公司不能再"假装不知道"训练数据中包含了哪些版权材料。

3. **GDPR 交叉适用**：如果训练数据中包含欧盟公民的个人数据（如社交媒体帖子、照片等），AI 公司还需遵守 GDPR——包括"被遗忘权"（用户有权要求删除其个人数据）和"数据最小化"原则。

这些规则的实施，意味着 AI 公司需要从"把所有能抓到的东西都喂给模型"转向"谨慎选择和管理训练数据"。

## 行业影响：谁最受伤？


<!-- AI_GEN: 版权规则收紧对不同玩家（大公司/开源社区/创作者）的影响矩阵 -->
版权规则的收紧对不同玩家的影响不对等：

**大公司**（OpenAI、Google、Meta）：虽然合规成本很高，但有足够的法务资源和技术能力来应对。而且他们已经在积极与内容版权方达成付费协议（Google 与新闻出版商的协议、OpenAI 与 Axel Springer 的协议等），为自己的训练数据"合法化"铺路。

**开源社区**：面临更大困难。开源模型（如 Llama 4、DeepSeek）使用的是公开可用的训练数据，但在新规则下，"公开可用"不等于"合法可用"。如果规则进一步收紧，开源模型的训练可能面临法律风险。

**内容创作者**：最终可能通过许可协议获得补偿——类似于音乐流媒体时代，平台向艺人支付版税。Getty Images 已经在 2025年推出了针对 AI 训练数据的许可计划。但单个创作者是否能获得公平的补偿，取决于集体谈判的博弈力。

## 中国的版权格局

在中国，AI 训练数据的版权问题也在升温。2026年初，北京互联网法院受理了首例"AI 训练数据版权侵权"案件——一位网络小说作者起诉某大模型公司在训练中未经授权使用其作品。

虽然中国的法律体系和欧美不同，但中国在 AI 监管上也在快速推进。网信办在 2025 年底发布了《生成式AI训练数据管理办法》征求意见稿，要求生成式 AI 服务提供者"使用合法来源的数据"。

## 结语

AI 训练数据的版权问题不会在 2026 年终结——它将在未来几年中持续发酵。但一个趋势是清晰的：**"免费爬虫时代"正在终结，AI 行业需要学会为创作付费。** 这本质上是一个"谁为 AI 的价值创造买单"的问题——是模型开发者独享收益，还是数据创作者也分一杯羹。

---
*参考来源：*
- [EU AI Act — Up-to-date developments and implementation timeline](https://artificialintelligenceact.eu/)
- [VerifyWise — Global AI Regulations Tracker 2026](https://verifywise.ai/global-ai-regulations)
- [Hung Yi Chen — AI Governance and Regulation 2026: A Complete Guide](https://www.hungyichen.com/en/insights/ai-governance-regulatory-landscape-2026)
- [MindFoundry — AI Regulations around the World (2026)](https://www.mindfoundry.ai/blog/ai-regulations-around-the-world)
