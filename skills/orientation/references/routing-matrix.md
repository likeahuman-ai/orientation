# Routing Matrix — Orientation Plugin

Decision tables for assigning participants to a track and level. These rules mirror `apps/claude-mastery/convex/workshopRouting.ts` exactly.

<!-- Mirrored from: apps/claude-mastery/convex/workshopRouting.ts -->
<!-- If you update this, update the backend too (or vice versa) -->

---

## Track Classification

Determines whether the participant goes on the **Developer** track or the **AI Assistant** track.

### Step 1: Check primary goal

If `primaryGoal === "workflow"` → **assistant** (confidence: 0.85). Done.

Otherwise, proceed to Step 2.

### Step 2: Keyword match on `buildDescription`

Scan the participant's build description (lowercase) for these keywords:

**ASSISTANT keywords:**
`automate`, `automation`, `workflow`, `pipeline`, `report`, `schedule`, `recurring`, `email sequence`, `scrape`, `monitor`, `digest`, `summarize daily`, `weekly report`

**DEV keywords:**
`app`, `platform`, `website`, `tool`, `dashboard`, `landing page`, `saas`, `marketplace`

### Step 3: Resolve

| Assistant keywords found | Dev keywords found | Result | Confidence |
|---|---|---|---|
| Yes | No | **assistant** | 0.9 |
| No | Yes | **dev** | 0.9 |
| Yes | Yes | **dev** (default when ambiguous) | 0.4 |
| No | No | **dev** (default when no signal) | 0.0 |

### Presenting low confidence

If confidence is 0.4 or lower, tell the participant:

> "Based on your description, I'm placing you on the **Developer track**, but your project has elements of both tracks. If you'd rather focus on automating a workflow instead of building an app, let me know and I'll switch you to the **AI Assistant track**."

If they want to switch, update the track. If they confirm dev, keep it.

---

## Level Assignment

Determines the difficulty level within the assigned track.

### Decision matrix

| Coding Experience | Terminal Comfort | Level |
|---|---|---|
| none | never | **fundamental** |
| none | scared | **fundamental** |
| beginner | never | **fundamental** |
| beginner | scared | **fundamental** |
| advanced | comfortable | **advanced** |
| advanced | advanced | **advanced** |
| expert | comfortable | **advanced** |
| expert | advanced | **advanced** |
| *anything else* | *anything else* | **intermediate** |

### Simplified rule

```
IF (codingExperience in [none, beginner]) AND (terminalComfort in [never, scared])
  → fundamental

IF (codingExperience in [advanced, expert]) AND (terminalComfort in [comfortable, advanced])
  → advanced

ELSE → intermediate
```

---

## Output Format

After routing, present the result to the participant:

**High confidence (>= 0.8):**
> "You're on the **[Track] track ([Level] level)**."

**Medium confidence (0.4 - 0.79):**
> "Based on your answers, I'm placing you on the **[Track] track ([Level] level)**. This seems like the best fit, but let me know if you'd prefer a different track."

**Low confidence (< 0.4):**
> "I'm placing you on the **[Track] track ([Level] level)** as a starting point. Your project could go either way — if this doesn't feel right as we go, we can switch."

---

## What the tracks mean (explain to participant)

**Developer track:** You'll build a web application — a real, working app you can deploy and share. The workshop guides you through planning, coding, and launching.

**AI Assistant track:** You'll build an automation — a workflow that runs tasks for you using Claude Code skills, hooks, and MCP servers. The workshop guides you through designing, prototyping, and connecting to real data.

**Fundamental level:** Step-by-step guidance. Every command explained. Training wheels on — perfect for first-timers.

**Intermediate level:** You know the basics. We'll give you structure but expect you to drive. The standard workshop experience.

**Advanced level:** You've built things before. We'll challenge you with professional patterns — TDD, iterative design, full deployment pipelines.
