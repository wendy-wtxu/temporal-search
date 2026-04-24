# temporal-search

A temporal-aware search enhancement skill for LLMs and AI Agents — making search results land in the right time window.

## The Problem

LLMs lack reliable temporal awareness in search scenarios. When a user asks about "League of Legends results", the model generates a search query without any year or season constraint, and the search engine returns SEO-heavy articles from years ago instead of this season's matches.

Common failure modes:
- **Missing time anchor** — "recent match" could mean today or 3 years ago to the model
- **No temporal constraint in queries** — "CES highlights" returns 2020 articles instead of the latest edition
- **Tense-intent mismatch** — "Where is the Olympics held?" — the user almost certainly means the next one, not a past one
- **Periodic event confusion** — "Black Friday deals" — which year's Black Friday?

## The Solution

`temporal-search` is a **pure reasoning middleware** that sits between the user's raw query and the actual search execution. It performs four steps:

1. **Time Anchoring** — Grounds "now" to a concrete date and derives relevant cycle anchors (current season, nearest event edition)
2. **Temporal Intent Classification** — Determines if the user wants realtime / recent / upcoming / periodic / historical / timeless information
3. **Query Time Augmentation** — Rewrites the search query with proper time constraints (year, month, season identifiers)
4. **Freshness Priority** — Provides ranking guidance for post-processing results by recency

## Supported Scenarios

| Scenario | Time Sensitivity | Example |
|----------|-----------------|---------|
| Esports & Sports | Very High | "NBA Finals", "League of Legends Worlds" |
| Breaking News | Very High | "earthquake latest" |
| Conferences & Expos | High | "CES", "WWDC", "MWC" |
| Policies & Regulations | High | "tax policy changes" |
| Product Launches | Medium-High | "latest iPhone" |
| Recurring Events | Medium | "Black Friday", "Super Bowl" |
| Project Procurement | Medium | "water treatment project bidding" |
| Encyclopedia / Concepts | Low | "how does photosynthesis work" |

## Usage

### As a Claude Skill
Place `SKILL.md` in your Claude Skills directory. The skill triggers automatically for time-sensitive queries.

### As a System Prompt Component
Include the content of `SKILL.md` in your LLM/Agent's system prompt to enable temporal-aware search behavior.

### As an Agent Middleware
Use the structured output format (see `SKILL.md` → Output Format) to programmatically augment queries before passing them to a search API.

## Files

```
temporal-search/
├── README.md          # This file
├── SKILL.md           # English version of the skill
├── SKILL_zh.md        # Chinese (中文) version of the skill
└── LICENSE            # MIT License
```

## Example

**User query:** "League of Legends results"

**Skill output:**
```json
{
  "temporal_intent": "periodic_current",
  "augmented_queries": [
    "League of Legends LPL 2026 Spring Split results standings",
    "LOL MSI 2026 schedule results",
    "League of Legends 2026 latest match results"
  ],
  "freshness_priority": {
    "level": "recent",
    "preferred_window": "last 3 months"
  }
}
```

Without the skill, a raw search for "League of Legends results" returns articles from 2020. With it, every query targets the current 2026 season.

## License

MIT

---

## 中文说明

# temporal-search

一个面向大模型（LLMs）与 AI Agent 的**时间感知搜索增强技能** —— 让搜索结果落在“正确的时间窗口”。

---

## 问题背景

大模型本身**缺乏时间感知能力**。  
例如，当用户提问类似“英雄联盟比赛结果”时，模型往往不会在搜索查询中加入年份或赛季约束，导致搜索引擎返回大量几年前的 SEO 内容，而不是当前赛季的结果。

常见问题包括：

- **缺少时间锚点**  
  “最近的比赛”对模型来说，可能是今天，也可能是三年前  

- **查询中没有时间约束**  
  “CES 精彩内容”可能返回 2020 年的文章，而不是最新一届  

- **时态与意图不匹配**  
  “奥运会在哪里举办？”用户几乎肯定想问下一届，而不是历史信息  

- **周期性事件混淆**  
  “黑色星期五优惠”——到底是哪一年的？

---

## 解决方案

`temporal-search` 是一个**纯推理型中间层（reasoning middleware）**，位于用户原始查询与实际搜索执行之间，核心执行四个步骤：

1. **时间锚定（Time Anchoring）**  
   将“当前时间”具体化为明确日期，并推导相关周期锚点（当前赛季、最近一届活动等）

2. **时间意图识别（Temporal Intent Classification）**  
   判断用户需求属于：实时 / 最近 / 即将发生 / 周期性 / 历史 / 常识类

3. **查询时间增强（Query Time Augmentation）**  
   自动为搜索语句补充时间约束（年份、月份、赛季标识等）

4. **新鲜度优先级（Freshness Priority）**  
   为搜索结果排序提供“时间优先”策略指导

---

## 适用场景

| 场景 | 时间敏感度 | 示例 |
|------|------------|------|
| 电竞 / 体育赛事 | 极高 | “NBA 总决赛”、“英雄联盟全球总决赛” |
| 突发新闻 | 极高 | “地震 最新消息” |
| 展会 / 发布会 | 高 | “CES”、“WWDC”、“MWC” |
| 政策法规 | 高 | “税收政策变化” |
| 产品发布 | 中高 | “最新 iPhone” |
| 周期性事件 | 中 | “黑色星期五”、“超级碗” |
| 项目招标 | 中 | “水处理项目招标” |
| 百科 / 原理类 | 低 | “光合作用原理” |

---

## 使用方式

### 作为 Claude Skill 使用
将 `SKILL.md` 放入 Claude Skills 目录中，技能会在检测到时间敏感问题时自动触发。

### 作为系统提示词组件
将 `SKILL.md` 内容加入到 LLM / Agent 的 system prompt 中，以增强时间感知搜索能力。

### 作为 Agent 中间层
使用结构化输出（见 `SKILL.md` → Output Format），在调用搜索 API 前对查询进行程序化增强。

---

## 文件结构

temporal-search/
├── README.md          # 当前说明文件
├── SKILL.md           # 英文版技能定义
├── SKILL_zh.md        # 中文版技能定义
└── LICENSE            # MIT 许可证

---

## 示例

**用户输入：**  
"英雄联盟比赛结果"

**技能输出：**
```json
{
  "temporal_intent": "periodic_current",
  "augmented_queries": [
    "英雄联盟 LPL 2026 春季赛 战绩 积分榜",
    "LOL MSI 2026 赛程 比赛结果",
    "英雄联盟 2026 最新比赛结果"
  ],
  "freshness_priority": {
    "level": "recent",
    "preferred_window": "最近 3 个月"
  }
}

如果没有该技能，直接搜索“英雄联盟比赛结果”可能返回 2020 年的旧内容。
引入该技能后，所有查询都会自动指向当前 2026 赛季。

许可证

MIT
