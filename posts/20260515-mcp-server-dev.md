---
title: "学会这一招，任何API都能变成Claude Code的工具——MCP Server开发实战"
date: 2026-05-15T12:00:00+08:00
categories: "技术分享"
id: "20260515"
cover: "/images/posts/mcp-dev.png"
tags:
  - MCP
  - Claude Code
  - Skill
  - AI
  - 教程
  - TypeScript
  - 2026
recommend: true
top: false
hide: false
author: "YEYUbaka"
description: "看了那么多MCP和Skill的介绍文章，你有没有想过自己写一个？这篇文章用完全免费的天气API作为案例，带你从零开发一个MCP服务器，并把它接入Claude Code。读完你会发现：原来开发一个AI工具，真的没你想的那么难。"
showToc: true
---

:::note{type="success"}
很多同学看完 MCP 和 Skill 的概念文之后都有一个共同的疑问：**"道理我都懂，但到底怎么自己写一个？"** 这篇文章就是用最接地气的方式回答这个问题。我选了一个完全免费、无需 API Key 的天气查询服务作为目标，带你一步步完成：读懂 API → 写 MCP Server → 接入 Claude Code → 让 AI 自动查天气。每一步都有代码、有解释、有踩坑记录，包你能跑通。
:::

## 一、先搞清楚我们要做什么

> 与其在概念里打转，不如直接看效果。

### 1.1 最终效果

写完这套 MCP Server 之后，你在 Claude Code 里就能这样用：

```
你：北京现在天气怎么样？
Claude Code：[调用天气工具]
→ 北京当前温度 22.3°C，体感 22.5°C，湿度 59%，大部晴朗

你：那上海呢？未来三天会下雨吗？
Claude Code：[调用预报工具]
→ 上海未来三天都是晴天，27-28°C，降雨概率很低
```

AI 不再只能"聊天气"，而是真的"查天气"。

### 1.2 为什么选"天气 API"作为教学案例

这就像学做菜，第一道菜肯定是番茄炒蛋，而不是佛跳墙。

| 选择标准 | 天气 API（Open-Meteo） | 其他 API（GitHub / 数据库等） |
| :--- | :--- | :--- |
| 是否需要注册 | 不需要 | 大多数需要账号 |
| 是否需要 API Key | 不需要 | 基本都需要 |
| 理解难度 | 一看就懂（温度、湿度、风速） | 需要懂领域知识 |
| 即时反馈 | 换个城市就有不同结果 | 写操作有风险 |
| 服务稳定性 | 开源项目，全球 CDN | 依赖商业服务 |

我们的目标是学会"怎么写 MCP Server"这个通用技能，而不是"怎么对接某个特定 API"。选一个零门槛的服务，你就能把全部注意力放在核心流程上。

### 1.3 你需要准备什么

- Node.js 环境（v18 或以上）
- Claude Code 已安装
- 一个文本编辑器
- 大概 15 分钟时间

### 1.4 整体流程

一张图说清楚整件事的链路：

```
Open-Meteo API（天气数据源）
        ↓
  MCP Server（把 API "翻译"成 AI 能调用的工具）
        ↓
  配置接入 Claude Code
        ↓
  AI 对话中自动查天气
```

> 如果说 API 是外国厨师的菜谱，那 MCP Server 就是中文翻译 + 厨房操作台。AI 通过它就能"做"出这道菜。

---

## 二、读懂 API：MCP Server 的前置功课

> 写 MCP Server 就像盖房子，而读 API 文档就是打地基。地基歪了，后面全白搭。

### 2.1 认识 Open-Meteo

[Open-Meteo](https://open-meteo.com/) 是一个完全免费、开源的世界天气预报 API。它的特点用一句话就能说完：**不用注册、不用 API Key、HTTPS 直接访问、全球覆盖**。

我们用到两个接口：
- **Geocoding API**：把"北京"翻译成经纬度（`39.9, 116.4`）
- **Forecast API**：根据经纬度返回天气数据

### 2.2 核心参数速览

天气预报接口的关键参数：

| 参数 | 含义 | 示例 |
| :--- | :--- | :--- |
| `latitude` | 纬度 | `39.9042`（北京） |
| `longitude` | 经度 | `116.4074`（北京） |
| `current` | 要获取的当前天气字段 | `temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code` |
| `daily` | 每日预报字段 | `temperature_2m_max,temperature_2m_min,precipitation_probability_max` |
| `timezone` | 时区 | `Asia/Shanghai`（推荐 `auto` 自动识别） |
| `forecast_days` | 预报天数 | `3`（默认 7） |

这些参数够你应对大部分场景了。入门先别贪多，把核心参数搞透再说。

### 2.3 浏览器控制台调试：最直观的 API 学习法

按 F12 打开开发者工具，切到 Console 标签，直接粘贴运行：

```javascript
// 直接复制到浏览器控制台就能看到结果
fetch('https://api.open-meteo.com/v1/forecast?latitude=39.9042&longitude=116.4074&current=temperature_2m&timezone=auto')
  .then(r => r.json())
  .then(d => console.log('北京当前温度:', d.current.temperature_2m + '°C'))
```

这个东西我愿称之为"工地上的皮尺"——随时量，随时看，所见即所得，比 Postman 还快。

### 2.4 我们要实现的三个核心功能

不贪多，文章聚焦三个最实用的功能：

1. **城市名转坐标**（Geocoding）：用户说"北京"，自动转成经纬度
2. **查当前天气**：温度、湿度、风速、天气状况
3. **查未来几天预报**：每日最高/最低温度、降雨概率

这三个功能覆盖了"即时查询"和"出行规划"两大场景，学完就能用。

---

## 三、动手写 MCP Server：把 API "翻译"成 AI 听得懂的工具

> 如果说 API 是厨师的菜谱，那 MCP Server 就是翻译 + 厨房操作台。AI 通过它就能"做"出天气查询这道菜。

### 3.1 项目初始化

```bash
mkdir weather-mcp-server
cd weather-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node
```

然后创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src"]
}
```

在 `package.json` 里加一句 `"type": "module"`，这样才能用 ES Module 的 import。

### 3.2 MCP Server 的骨架

一个 MCP Server 由三个核心组件构成：

| 组件 | 作用 | 类比 |
| :--- | :--- | :--- |
| **McpServer 实例** | 负责通信协议 | 餐厅服务员 |
| **Tool 定义** | 告诉 AI "我能做什么" | 菜单 |
| **Tool Handler** | 实际执行 API 调用 | 后厨 |

我们先搭骨架，再逐个填肉。

### 3.3 核心代码：完整实现

打开 `src/index.ts`，下面是完整代码：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// ── 基础配置 ──
const GEOCODING_BASE = "https://geocoding-api.open-meteo.com/v1";
const FORECAST_BASE = "https://api.open-meteo.com";

// ── 城市名 → 坐标 ──
async function geocode(city: string) {
  const url = `${GEOCODING_BASE}/search?name=${encodeURIComponent(city)}&count=3&language=zh&format=json`;
  const res = await fetch(url);
  if (!res.ok) return null;
  const data = await res.json() as { results?: Array<{ name: string; latitude: number; longitude: number; country: string; admin1?: string }> };
  return data.results?.[0] ?? null;
}

// ── 创建 Server ──
const server = new McpServer(
  { name: "weather-server", version: "1.0.0" },
  {
    capabilities: {},
    instructions: "使用此服务器查询天气信息。先调用 geocode 将城市名转为坐标，再调用天气工具获取天气数据。"
  }
);

// ── Tool 1：城市名转坐标 ──
server.registerTool(
  "geocode",
  {
    description: "将城市名称转换为经纬度坐标",
    inputSchema: z.object({
      city: z.string().describe("城市名称，如'北京'、'上海'")
    })
  },
  async ({ city }) => {
    const result = await geocode(city);
    if (!result) return { content: [{ type: "text", text: `未找到城市：${city}` }] };
    const region = result.admin1 ? `${result.country} ${result.admin1}` : result.country;
    return { content: [{ type: "text", text: `${result.name}：纬度 ${result.latitude}，经度 ${result.longitude}（${region}）` }] };
  }
);

// ── Tool 2：查询当前天气 ──
server.registerTool(
  "get_current_weather",
  {
    description: "查询指定城市当前的天气情况",
    inputSchema: z.object({
      city: z.string().describe("城市名称，如'北京'、'上海'")
    })
  },
  async ({ city }) => {
    const geo = await geocode(city);
    if (!geo) return { content: [{ type: "text", text: `未找到城市：${city}` }] };

    const url = `${FORECAST_BASE}/v1/forecast?latitude=${geo.latitude}&longitude=${geo.longitude}&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m&timezone=auto`;
    const res = await fetch(url);
    const data = await res.json() as { current?: Record<string, number> };

    const c = data.current;
    if (!c) return { content: [{ type: "text", text: `未能获取 ${city} 的天气数据` }] };

    const weatherMap: Record<number, string> = {
      0: "☀️ 晴天", 1: "🌤 大部晴朗", 2: "⛅ 多云", 3: "☁️ 阴天",
      45: "🌫 有雾", 51: "🌧 小毛毛雨", 61: "🌧 小雨", 63: "🌧 中雨",
      65: "🌧 大雨", 71: "❄️ 小雪", 73: "❄️ 中雪", 75: "❄️ 大雪",
      80: "🌧 阵雨", 95: "⛈️ 雷暴", 96: "⛈️ 冰雹雷暴", 99: "⛈️ 大冰雹雷暴"
    };

    return {
      content: [{
        type: "text",
        text: [
          `${geo.name} 当前天气：`,
          `  🌡 温度：${c.temperature_2m}°C（体感 ${c.apparent_temperature}°C）`,
          `  💧 湿度：${c.relative_humidity_2m}%`,
          `  🌬 风速：${c.wind_speed_10m} km/h`,
          `  ☁ 天气：${weatherMap[c.weather_code] || `代码 ${c.weather_code}`}`
        ].join("\n")
      }]
    };
  }
);

// ── Tool 3：天气预报 ──
server.registerTool(
  "get_weather_forecast",
  {
    description: "查询指定城市未来几天的天气预报",
    inputSchema: z.object({
      city: z.string().describe("城市名称"),
      days: z.number().default(3).describe("预报天数，1-7")
    })
  },
  async ({ city, days }) => {
    const geo = await geocode(city);
    if (!geo) return { content: [{ type: "text", text: `未找到城市：${city}` }] };

    const d = Math.min(Math.max(days ?? 3, 1), 7);
    const url = `${FORECAST_BASE}/v1/forecast?latitude=${geo.latitude}&longitude=${geo.longitude}&daily=temperature_2m_max,temperature_2m_min,precipitation_probability_max,weather_code&timezone=auto&forecast_days=${d}`;
    const res = await fetch(url);
    const data = await res.json() as { daily?: { time: string[]; temperature_2m_max: number[]; temperature_2m_min: number[]; precipitation_probability_max: number[]; weather_code: number[] } };

    if (!data.daily) return { content: [{ type: "text", text: `未能获取 ${city} 的预报数据` }] };

    const weatherSimple = (code: number): string => {
      if (code <= 3) return "晴";
      if (code <= 48) return "雾/霾";
      if (code <= 67) return "雨";
      if (code <= 77) return "雪";
      if (code <= 82) return "阵雨";
      if (code <= 86) return "阵雪";
      return "雷暴";
    };

    const lines = [`${geo.name} 未来 ${d} 天天气预报：`];
    for (let i = 0; i < d; i++) {
      const icon = data.daily.precipitation_probability_max[i] > 50 ? "🌧" : "☀";
      lines.push(`  ${data.daily.time[i]} ${icon} ${weatherSimple(data.daily.weather_code[i])} | 高温 ${data.daily.temperature_2m_max[i]}°C / 低温 ${data.daily.temperature_2m_min[i]}°C | 降雨概率 ${data.daily.precipitation_probability_max[i]}%`);
    }

    return { content: [{ type: "text", text: lines.join("\n") }] };
  }
);

// ── 启动 ──
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP Server running on stdio");
}

main().catch((err) => {
  console.error("启动失败:", err);
  process.exit(1);
});
```

代码总共不到 120 行，但结构很清晰：一个 geocode 辅助函数 + 三个 registerTool 调用 + 一个 main 启动入口。

### 3.4 几个关键细节

**关于 import 路径**：注意导入路径必须带 `.js` 后缀，不然 Node.js ESM 会报 `ERR_MODULE_NOT_FOUND`。这是 MCP SDK 的 wildcard export 机制决定的，是很容易踩的坑。

**关于 `FORECAST_BASE`**：基础 URL 是 `https://api.open-meteo.com`，不是 `https://api.open-meteo.com/v1`。我当初写成了后者，结果拼出来的 URL 变成了 `api.open-meteo.com/v1/v1/forecast`，查了半天才发现是重复了路径。

**关于天气代码**：Open-Meteo 用的是 [WMO 天气代码](https://www.nodc.noaa.gov/archive/arc0021/0002199/1.1/data/0-data/HTML/WMO-CODE/WMO4677.HTM)，我挑了几个最常用的做了映射。你想更精确的话可以去看官方文档补全。

### 3.5 构建和测试

```bash
# 构建
npx tsc

# 快速验证：手动发送一个初始化请求
printf '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}\n{"jsonrpc":"2.0","method":"notifications/initialized"}\n{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}\n' | node dist/index.js
```

如果看到返回了 3 个 Tool 的定义，说明你的 MCP Server 准备好了。

### 3.6 接入 Claude Code

在项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["E:/AI_projects/weather-mcp-server/dist/index.js"]
    }
  }
}
```

路径用你的实际项目路径。重启 Claude Code 后，天气工具就会自动加载。

---

## 四、踩坑与排查

> 代码写完不报错，只是过了第一关；真正能稳定跑起来的，都是踩过坑的。

### 4.1 常踩的坑

| 现象 | 原因 | 解决 |
| :--- | :--- | :--- |
| `ERR_MODULE_NOT_FOUND` | import 路径缺 `.js` 后缀 | 加上 `.js`：`from ".../mcp.js"` |
| API 返回 `"Not Found"` | 基础 URL 重复了 `/v1` | 检查 `FORECAST_BASE` 是否已包含版本路径 |
| 中文城市名查不到 | Geocoding API 未加 `language=zh` | URL 里加 `&language=zh` |
| Claude Code 不显示工具 | `.mcp.json` 语法错误 | `jq . .mcp.json` 检查 JSON 是否合法 |
| 工具加载了但调用失败 | MCP Server 进程没启动 | 检查 `dist/index.js` 路径是否正确 |

### 4.2 排查心法

遇到问题时，最有效的调试方法是：

1. **先在浏览器控制台用 fetch 单独测 API**——看是服务器端问题还是你代码的问题
2. **用 `console.error` 打日志**——记住 MCP 用 stdio 通信，日志必须打到 stderr，不然会干扰协议
3. **手动 pipe 测试请求给 server**——绕过 Claude Code，直接看 server 是否正常响应

> 排查的顺序永远是：先确认 API 正常 → 再确认 server 正常 → 最后排查 Claude Code 配置。

---

## 五、从 MCP 到 Skill：让 AI 更"懂"你的工具

> MCP Server 让 AI 有了"手"（能调用 API），但 Skill 让 AI 有了"脑子"（知道什么时候该用、怎么组合使用）。这就好比你会切菜不等于你会做菜——Skill 就是菜谱。

### 5.1 什么时候需要写一个 Skill

| 场景 | 只用 MCP Server | MCP + Skill |
| :--- | :--- | :--- |
| 问"北京天气" | ✅ AI 知道调用工具 | ✅ 一样 |
| 问"我这周末出差，帮我看看天气" | ❌ AI 不知道该用哪个工具 | ✅ Skill 引导多工具组合 |
| 问"对比 5 个城市，推荐最适合明天出行的" | ❌ AI 可能只查一个 | ✅ Skill 自动多轮查询 + 对比 |

简单来说：**MCP Server 解决了"有没有工具"的问题，Skill 解决了"怎么用工具"的问题**。

### 5.2 写一个简单的天气 Skill

在你的项目 `.claude/skills/` 目录下创建 `weather-reporter.md`：

```markdown
## 触发条件
当用户询问天气、出行建议、气温相关信息时激活。

## 工作流程
1. 先用 geocode 将城市名转为坐标
2. 如果是"当前/现在就"，调用 get_current_weather
3. 如果是"明天/周末/下周"，调用 get_weather_forecast
4. 如果涉及多个城市对比，逐个查询后汇总
5. 用自然语言回答，突出关键信息

## 输出规范
- 默认用摄氏度
- 如果降雨概率 > 50%，提醒带伞
- 如果温度 > 30°C，提醒防暑
- 如果温度 < 10°C，提醒保暖
```

这个 Skill 本质上是一份"使用说明书"，它告诉 AI 在什么场景下应该调用哪个工具、按什么顺序调、输出时要注意什么。

### 5.3 效果对比

不带 Skill 时，AI 可能会直接给你 JSON 原始数据；带上 Skill 之后，它会：

- 自动判断你是要当前天气还是预报
- 把多轮查询结果整合成一段顺畅的回复
- 主动加上穿衣建议和出行提醒

---

## 六、社区已有轮子：别重复造方向盘

> 就像买车不用先学造车，很多 MCP Server 别人已经写好了，直接用就行。

### 6.1 推荐已有的 MCP Server

| 项目 | 功能 | 适用场景 |
| :--- | :--- | :--- |
| [puppeteer MCP Server](https://github.com/modelcontextprotocol/servers) | 浏览器自动化 | 网页抓取、截图、表单填写 |
| [filesystem MCP Server](https://github.com/modelcontextprotocol/servers) | 文件系统操作 | 读写本地文件 |
| [brave-search MCP Server](https://github.com/modelcontextprotocol/servers) | 网络搜索 | 实时信息检索 |
| [sequential-thinking MCP Server](https://github.com/modelcontextprotocol/servers) | 强化推理 | 复杂逻辑分析 |

### 6.2 怎么判断"该自己写"还是"用现成的"

- **用现成的**：API 是主流服务（GitHub、Slack、数据库等）→ 直接搜 "MCP Server + 名字"
- **自己写**：内部系统 API、小众服务、需要定制业务逻辑 → 参考本文的套路手写

多数情况下，一个 100 行左右的 MCP Server 就够用了，核心代码量其实很少。

---

## 七、总结

> 回顾一下，我们今天做了什么？

1. 读懂了一个免费天气 API（Open-Meteo）
2. 用不到 120 行 TypeScript 写了一个 MCP Server（三个 Tool）
3. 把它接入了 Claude Code
4. 让 AI 能自动查天气、做预报
5. 还顺带写了个 Skill 让它更聪明

**最关键的一点：这套方法论适用于任何 API。**

新闻 API、股票 API、物流 API、翻译 API……学会今天的内容，你就拥有了"让 AI 连接万物的能力"。每次遇到一个新的 API，你只需要三步：
- 读懂它的参数和返回格式
- 用 `registerTool` 定义工具
- 在 Handler 里调用 fetch

其他都是照搬模板。

> AI 时代的核心竞争力，不是你会用多少 AI 工具，而是你能否把身边的任何一个 API，都变成 AI 能调用的能力。

---

**参考资料**：

- [Open-Meteo 官方文档](https://open-meteo.com/en/docs)
- [MCP 官方文档](https://modelcontextprotocol.io)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Claude Code MCP 集成文档](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [WMO 天气代码对照表](https://www.nodc.noaa.gov/archive/arc0021/0002199/1.1/data/0-data/HTML/WMO-CODE/WMO4677.HTM)
