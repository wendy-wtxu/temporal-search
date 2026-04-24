---
name: temporal-search
description: >
  Temporal-aware search enhancement skill.
  Before an LLM/Agent executes a search, this skill parses the user's query for temporal intent
  and augments it with proper time constraints, ensuring search results land in the user's expected
  time window. Applicable scenarios include but are not limited to: esports & sports (League of Legends,
  NBA, World Cup, Olympics), conferences & expos (CES, MWC, WWDC), breaking news, policies & regulations,
  product launches, recurring events (Black Friday, Super Bowl, Singles' Day), and government procurement.
  Triggers when the query contains any time-sensitive information — keywords like "latest", "recent",
  "last season", "next edition", "this year", "current season" — or when the query subject itself
  has strong temporal attributes (tournaments, news, launches, policies).
  Even if no explicit time words are present, this skill should trigger as long as the topic
  belongs to a time-sensitive category.
---

# Temporal Search Enhancer

## Core Concept

LLMs lack an intrinsic sense of time — they don't know what "now" is, nor do they understand which specific time window "recent" means to the user. This causes search queries to lack temporal constraints, leading search engines to return outdated or misaligned results.

This skill acts as a **pure reasoning middleware** between the user's raw query and search execution, injecting temporal reasoning so every search query carries the correct time anchor.

---

## Execution Flow

Upon receiving a user query, execute the following four steps in order. Each step's output feeds into the next.

### Step 1: Time Anchoring

Make "now" concrete. Based on the system-provided current date, establish the following context variables:

```
current_date: 2026-04-25        # Exact current date
current_year: 2026
current_quarter: Q2
current_month: April
current_weekday: Saturday
```

Also derive **cycle anchors** relevant to the query topic, for example:
- Sports: Which season are we in? (e.g., "2025-2026 season", "2026 Spring Split")
- Conferences: When was the most recent edition? When is the next?
- Recurring events: When was the last instance? When is the next?

Cycle anchors don't need day-level precision — year or season level is sufficient to guide subsequent steps.

### Step 2: Temporal Intent Classification

Determine the temporal direction of the user's query. Classify into the following categories:

| Intent Type | Meaning | Typical Signal Words | Example |
|------------|---------|---------------------|---------|
| **realtime** | Needs current-moment / same-day data | "score", "live", "right now" | "NBA game score" |
| **recent** | Needs recent (retrospective) data | "latest", "recent", "new" | "latest AI news" |
| **upcoming** | Needs future event information | "next", "when is", "upcoming" | "next Olympics location" |
| **periodic_current** | Locate the current/nearest cycle instance | The periodic event name itself | "Black Friday deals", "Spring Split" |
| **historical** | Explicitly points to a past time | "last year", "in 2020", "previous" | "2022 World Cup winner" |
| **timeless** | No temporal constraint needed | No time signals | "what is quantum computing" |

**Classification Rules:**
- If the query contains explicit time words, use the corresponding type directly
- If no explicit time words exist but the subject has strong temporal attributes (tournaments, news, policies, etc.), default to `periodic_current` or `recent`
- Only classify as `timeless` when the subject is genuinely time-independent (pure concepts, definitions)
- When in doubt, prefer adding a time constraint — the cost of one extra year keyword is far less than returning outdated results

### Step 3: Query Time Augmentation

Based on the anchors from Step 1 and the intent from Step 2, rewrite or supplement the user's raw query.

**Augmentation Strategies:**

| Intent Type | Augmentation Method | Example (assuming current date: April 2026) |
|------------|--------------------|--------------------------------------------|
| realtime | Add "today" / "live" + exact date | "LOL match" → "League of Legends match results April 25 2026" |
| recent | Add recent time range | "AI news" → "AI news April 2026" |
| upcoming | Add "upcoming" + year | "Olympics" → "2028 Olympics schedule venue" |
| periodic_current | Add current cycle identifier | "League of Legends" → "League of Legends 2026 Spring Split / MSI 2026" |
| historical | Preserve user-specified time | "2022 World Cup winner" → keep as-is |
| timeless | No time constraint | "quantum computing principles" → keep as-is |

**Augmentation Principles:**
- Generate 2-3 complementary augmented queries covering different angles (e.g., results, schedule, standings)
- Use search-engine-friendly formats for time identifiers: numeric years (2026), months as numbers or words
- Preserve core keywords from the user's original query — do not over-rewrite the semantics
- For periodic tournaments, use both common names and official event names (e.g., "League of Legends" + "LPL" + "MSI")

### Step 4: Freshness Priority

Provide time-dimension ranking guidance for post-processing search results:

| Intent Type | Freshness Strategy |
|------------|-------------------|
| realtime | Prioritize content from last 24 hours, then last week; do not discard older background context |
| recent | Prioritize last month, then last 3 months |
| upcoming | Prioritize content containing future dates |
| periodic_current | Prioritize content from the current cycle (e.g., current season) |
| historical | Prioritize content matching the specified time |
| timeless | No time preference; rank by relevance |

Note: This is **priority ranking**, not hard filtering. Even in breaking news scenarios, do not discard background content older than 24 hours — just lower its ranking weight.

---

## Output Format

After completing the four-step analysis, output the following structured result (for downstream search agents or systems):

```json
{
  "original_query": "user's raw query",
  "current_date": "2026-04-25",
  "temporal_intent": "periodic_current",
  "time_anchor": {
    "description": "2026 season Spring Split / MSI phase",
    "primary_period": "2026",
    "sub_period": "Spring Split / MSI"
  },
  "augmented_queries": [
    "League of Legends LPL 2026 Spring Split results standings",
    "LOL MSI 2026 schedule results",
    "League of Legends 2026 latest match results"
  ],
  "freshness_priority": {
    "level": "recent",
    "preferred_window": "last 3 months",
    "fallback_window": "full year 2026"
  },
  "reasoning": "The query 'League of Legends results' has no explicit time reference, but esports is a strongly time-sensitive topic..."
}
```

---

## Edge Case Handling

### Cross-Year / Cross-Season Boundaries
Tournament seasons often span calendar years (e.g., 2025-2026 season). When the current date falls mid-season, cover both the start and end of the season window.

### Multi-Tier Event Systems
For ecosystems with multiple event tiers (e.g., League of Legends: LPL/LCK/LEC regional leagues → MSI → Worlds), determine which tier is currently active or most recently concluded based on the current date, and prioritize querying that tier.

### Regional Differences
The same time-related term may refer to different events in different regions. For example, "the league" in a Chinese context likely means LPL, while in Korea it likely means LCK. Use `user_locale` to disambiguate.

### Ambiguous Intent
When it's unclear whether the user is looking back or forward (e.g., "World Cup" in a non-tournament year), generate queries for both the previous and upcoming editions, letting search results provide natural coverage.

---

## Integration with Search Tools

This skill's output can directly guide the following search behaviors:
1. Feed the `augmented_queries` into the search tool one by one
2. Rank returned results according to `freshness_priority`
3. When answering the user, annotate the time point of the information to help them assess timeliness

If the search tool supports time-range filter parameters (e.g., Google's `tbs=qdr:m`), use them preferentially; otherwise, constrain implicitly via query keywords.
