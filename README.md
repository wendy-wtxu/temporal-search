# temporal-search

A temporal-aware search enhancement skill for LLMs and AI Agents — making search results land in the right time window.

## The Problem

LLMs have no sense of time. When a user asks about "League of Legends results", the model generates a search query without any year or season constraint, and the search engine returns SEO-heavy articles from years ago instead of this season's matches.

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

**temporal-search** 是一个时间感知搜索增强 Skill，用于解决大模型在搜索时缺乏时间观念的问题。

当用户查询涉及赛事、新闻、展会、政策等有强时间属性的主题时，本 Skill 会自动在搜索query中注入正确的时间约束（年份、赛季、月份等），确保搜索结果命中用户期望的时间窗口。

中文版 Skill 文件见 [SKILL_zh.md](./SKILL_zh.md)。
