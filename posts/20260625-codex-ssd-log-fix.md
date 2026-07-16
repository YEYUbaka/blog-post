---
title: "Codex 日志把 SSD 写爆了？一条 SQLite 触发器应急止血"
date: 2026-06-25T23:50:00+08:00
categories: "技术分享"
id: "20260625"
cover: "/images/posts/codex-ssd-log-fix/cover.webp"
tags:
  - Codex
  - SSD
  - SQLite
  - 踩坑记录
  - 2026
description: "Codex 近期被曝出 logs_2.sqlite TRACE 日志导致 SSD 写入放大问题，15秒写入近3000条记录，WAL 膨胀至46MB。本文提供一套应急修复方案：检查文件大小、备份、创建 SQLite 触发器禁止新写入、截断 WAL 并复查，避免继续磨损 SSD。"
hide: false
top: false
recommend: false
author: "YEYUbaka"
showToc: true
---

::vhMusic{id="2029892284"}

:::note{type="warning"}
**如果你在用 Codex 写代码**，突然发现硬盘灯狂闪、SSD 健康度暴跌、系统盘写入量异常——那么你大概率被 Codex 的 `logs_2.sqlite` TRACE 日志写盘 bug 打中了。

社区实测数据：一块 1TB SSD 在 21 天内累积了约 37TB 写入量，折合每天 1.76TB。按典型消费级 NVMe SSD 600 TBW 耐久度算，不到一年就能把一块新盘写废。

这篇文章给你一套**可立即执行的应急止血方案**：先查后备、建触发器、截 WAL、等 30 秒复查。全程 5 分钟，不折腾。
:::

## 参考来源

- [Codex SSD Logging Bug — Codexer 社区讨论](https://codexer.com/posts/2026-06-23-codex-ssd-logging-bug/)
- [Codex 导致 SSD 写入放大分析 — CSDN](https://blog.csdn.net/chen695969/article/details/162230776)
- [Codex 日志疯狂写入硬盘问题 — EET China](https://www.eet-china.com/mp/a504606.html)

## 一、Codex 自己说了什么

在动手之前，先看一眼 Codex 自己对这件事的判断——给它把问题描述清楚，它就能定位到自己的日志子系统，并给出和本文方案一致的修复建议。

![Codex 自己确认 logs_2.sqlite 日志写入 bug 并给出修复建议](/images/posts/codex-ssd-log-fix/codex-fix-dialog.png)

*▲ Codex 自己确认 logs_2.sqlite TRACE 日志导致 SSD 写入放大，并给出了应急修复方案*

## 二、问题现象

### 定位问题文件

```bash
# 查看日志目录
ls -lah ~/.codex/
```

重点关注这些文件：

| 文件 | 作用 | 异常表现 |
|------|------|----------|
| `logs_2.sqlite` | Codex 日志主库 | 快速膨胀，可达数百 MB |
| `logs_2.sqlite-wal` | Write-Ahead Log | 持续增长，频繁刷盘 |
| `logs_2.sqlite-shm` | 共享内存索引 | 通常较小，随 WAL 波动 |

### 快速检测 WAL 状态

```bash
# 进入 SQLite 命令行
sqlite3 ~/.codex/logs_2.sqlite

-- 查看当前 WAL 日志模式（应该显示 wal）
PRAGMA journal_mode;

-- 查看日志表的写入规模
SELECT COUNT(*) FROM logs;
SELECT MAX(id) FROM logs;
```

社区实测数据：**15 秒内 `MAX(id)` 增长 2,978 条**，WAL 文件膨胀至 **46,601,352 字节**（约 44.5 MB）。按这个速度，一小时能写入约 70 万条 TRACE 日志。

:::note{type="info"}
**为什么看着文件不大，SSD 却在被写爆？**

这就是"写入放大"（Write Amplification）的核心。SQLite 每次 INSERT 不仅写数据，还触发页分裂、B-tree 重组、WAL 刷新等操作。加上 TRACE 日志在做高频"插入—删除—插入"循环，每次操作都可能触发物理写入。**表面文件只有几百 MB，底层 NAND 实际写入量可能是文件大小的几十倍甚至上百倍。**
:::

## 三、根因简述

原因归结为三点叠加：

1. **Codex 日志子系统默认以 TRACE 级别运行**，且该设置**无法通过环境变量调低**——`RUST_LOG` 被完全忽略。社区分析指出约 **71% 的写入属于 TRACE 级噪音日志**，对普通用户毫无意义。

2. **SQLite 的 WAL（Write-Ahead Logging）机制**：每条 INSERT 先写入 WAL 文件，再通过 checkpoint 合并到主库。高频写入下 WAL 几乎不停刷盘。

3. **写入放大**：TRACE → SQLite INSERT → 页分裂 → WAL 刷盘 → checkpoint → 主库更新 → WAL 循环。一条日志触发多层物理写入。

一个直观的理解：

```
一条 TRACE 日志 = 1 条 INSERT
  → 页已满？页分裂（写 2~3 个新页）
  → WAL 追加一帧
  → checkpoint 时把 WAL 帧合并回主库（再写一次）
  → 如果 VACUUM 被触发，重写整个数据库（灾难级放大）

结果：1 条日志 ≈ 多次物理写入
```

## 四、应急修复步骤

核心思路：**备份 → 阻止新写入 → 截断 WAL → 验证生效**。

### 4.1 检查当前状态

```bash
# 查看文件大小
ls -lah ~/.codex/logs_2.sqlite*

# 进入 SQLite 检查
sqlite3 ~/.codex/logs_2.sqlite <<EOF
PRAGMA journal_mode;
SELECT COUNT(*) AS total_rows FROM logs;
SELECT MAX(id) AS max_id FROM logs;
EOF
```

记下当前的 `total_rows` 和 `max_id`，后面验证时用。

### 4.2 备份数据库

```bash
# 备份主库和 WAL 文件
cp ~/.codex/logs_2.sqlite ~/.codex/logs_2.sqlite.bak
cp ~/.codex/logs_2.sqlite-wal ~/.codex/logs_2.sqlite-wal.bak 2>/dev/null

# 确认备份成功
ls -lah ~/.codex/logs_2.sqlite.bak
```

### 4.3 创建触发器禁止新写入

首先要确认日志表的表名（大概率是 `logs`）：

```bash
# 列出所有表
sqlite3 ~/.codex/logs_2.sqlite ".tables"
```

然后创建 `BEFORE INSERT` 触发器，用 `RAISE(IGNORE)` 静默丢弃所有新增写入：

```sql
-- 进入 SQLite
sqlite3 ~/.codex/logs_2.sqlite

-- 创建触发器：在每次 INSERT 之前触发，直接忽略
CREATE TRIGGER block_codex_log_insert
BEFORE INSERT ON logs
BEGIN
    SELECT RAISE(IGNORE);
END;

-- 确认触发器已创建
SELECT name, sql FROM sqlite_master WHERE type = 'trigger';
```

:::note{type="import"}
**为什么用 `RAISE(IGNORE)` 而不是 `RAISE(ABORT)`？**

`IGNORE` 静默跳过 INSERT 操作，Codex 不会收到任何错误反馈，照常运行。而 `ABORT` 会抛出异常回滚事务，可能引发 Codex 报错甚至崩溃。我们的目标是止血，不给它添新伤。
:::

### 4.4 截断 WAL 日志

**⚠️ 重点：不要执行 `VACUUM`！**

VACUUM 会完整重写整个数据库文件，在当前场景下等于手动触发一波几十倍写入放大，和我们的目标背道而驰。

正确的做法是用 `PRAGMA wal_checkpoint(TRUNCATE)`：

```sql
-- 在 sqlite3 中执行
PRAGMA wal_checkpoint(TRUNCATE);
```

这会做三件事：
1. 将 WAL 中所有未写回的帧合并到主库
2. 将 WAL 文件截断为 **0 字节**
3. 重置 WAL 写入序列号

执行完立即检查：

```bash
# 确认 WAL 文件已归零
ls -lah ~/.codex/logs_2.sqlite-wal
```

`TRUNCATE` 模式的 checkpoint 返回值含义：

```
busy  total  log  checkpointed
0     12345  0    12345
```

- `busy=0` → 无其他连接占用，checkpoint 顺利完成
- `log=0` → WAL 帧已清零
- `checkpointed` = `total` → 所有帧已写回

### 4.5 等待 30 秒并复查

```bash
# 等半分钟，期间不要操作 Codex（让它自然尝试写入）
sleep 30

# 复查
ls -lah ~/.codex/logs_2.sqlite*

# 再次检查日志表
sqlite3 ~/.codex/logs_2.sqlite <<EOF
SELECT COUNT(*) AS total_rows FROM logs;
SELECT MAX(id) AS max_id FROM logs;
EOF
```

## 五、验证触发器生效

把修复前的数据和 30 秒后的数据做个对比：

| 指标 | 修复前 | 30 秒后 | 状态 |
|------|--------|---------|------|
| `logs_2.sqlite` 大小 | 你的实测值 | 应与之前相近或略增 | ✅ stop |
| `logs_2.sqlite-wal` 大小 | 46 MB 量级 | **0 字节** | ✅ stop |
| `MAX(id)` | 2 千万左右 | **与修复前相同** | ✅ stop |
| `COUNT(*)` | 你的实测值 | **与修复前相同** | ✅ stop |

如果以上四个指标都停止增长，触发器就生效了。

你还可以主动验证触发器是否在工作——手动插一条试试：

```sql
sqlite3 ~/.codex/logs_2.sqlite "INSERT INTO logs DEFAULT VALUES;"
-- 不会报错，但也不会插入任何数据

-- 确认行数没变
SELECT COUNT(*) FROM logs;
```

:::note{type="success"}
**比符号链接方案好在哪？**

部分社区方案是把 `logs_2.sqlite` 软链到 `/tmp`。但 `/tmp` 默认也在 SSD 上（除非单独挂载为 tmpfs），写入放大还是发生在同一块盘上。触发器方案直接**从源头阻断 INSERT**，不产生任何新的物理写入，更彻底。
:::

## 六、风险提示

这个方案是**临时止血**，有几个需要注意的点：

1. **日志缺失**：触发器生效期间，Codex 的日志不会写入。如果需要排查其他问题，可以先删除触发器恢复正常日志：
   ```sql
   DROP TRIGGER block_codex_log_insert;
   ```

2. **日志级别无法调整**：目前 Codex 不响应 `RUST_LOG` 环境变量，降低日志级别的常规手段无效。只能等官方修复。

3. **重启 Codex 后检查**：确认 Codex 更新或重启后触发器是否仍然存在。SQLite 触发器存储在数据库文件中，只要 `logs_2.sqlite` 没被重建，触发器就在。

4. **不影响对话数据**：`logs_2.sqlite` 不包含任何用户对话记录，丢失这些日志对使用无影响。

5. **关注官方更新**：GitHub 社区已在跟进，一旦官方推送修复，记得移除触发器并更新版本。

## 七、总结

整套操作归纳为一张命令速查卡：

```bash
# 1. 检查
ls -lah ~/.codex/logs_2.sqlite*
sqlite3 ~/.codex/logs_2.sqlite "SELECT COUNT(*), MAX(id) FROM logs;"

# 2. 备份
cp ~/.codex/logs_2.sqlite ~/.codex/logs_2.sqlite.bak

# 3. 阻断
sqlite3 ~/.codex/logs_2.sqlite <<SQL
CREATE TRIGGER block_codex_log_insert BEFORE INSERT ON logs
BEGIN SELECT RAISE(IGNORE); END;
PRAGMA wal_checkpoint(TRUNCATE);
SQL

# 4. 等 30 秒后验证
sleep 30
ls -lah ~/.codex/logs_2.sqlite*
sqlite3 ~/.codex/logs_2.sqlite "SELECT COUNT(*), MAX(id) FROM logs;"
```

四个步骤：**检查 → 备份 → 阻断 → 验证**。全程 5 分钟，把 SSD 从 Codex 的 TRACE 日志轰炸中解救出来。

工具再智能，底层写盘失控，一样能把体验和硬件一起搞崩。记住这个教训：AI 工具不仅要看它的模型效果，日志、缓存、状态库这些"后台行为"同样值得关注。

---

*本文方案参考了 CSDN、Codexer 社区及 EET China 的讨论，致敬那些用 SSD 寿命踩坑的先驱者。期待 Codex 官方早日修了这个日志级别问题。*
