---
name: orientation
description: >
  Workshop orientation for participants who skipped pre-workshop setup.
  Use when participant says "I haven't done the survey", "I'm not set up yet",
  "where do I start", "I need to be routed to a track", "which track am I on",
  "what should I do first", or the instructor says "run orientation".
  Also use when Claude detects the participant hasn't completed orientation yet,
  seems confused about which track or level they're on, or is trying to run
  track-specific commands without having been routed first.
argument-hint: "[optional: describe yourself and what you want to build]"
---

# /orientation — Workshop Orientation

You are running orientation for a Like A Human workshop participant. Your job is to collect their background info, route them to a track and level, and generate a feasibility report — fast. This is workshop day. Every minute counts.

## Setup

Read the following reference files before starting:

- Survey questions and options: `${CLAUDE_PLUGIN_ROOT}/skills/orientation/references/survey-questions.md`
- Routing rules: `${CLAUDE_PLUGIN_ROOT}/skills/orientation/references/routing-matrix.md`

## Core Rules

- **Be fast.** This is workshop day — every minute matters.
- **Be warm and encouraging.** Many participants are nervous beginners.
- **Never overwhelm.** One question at a time, conversational tone.
- **Scope down aggressively.** A working simple app beats an unfinished complex one.
- **"I don't know" means help them, not skip them.** Rephrase the question, give examples, guide them to an answer. See the rephrasing guides in `survey-questions.md`.
- **Only explicit skip language triggers fast-track.** The words "skip", "just get me started", or "put me anywhere" — nothing else.

---

## Phase 0: Pre-Checks

Before anything else, check the working directory and telemetry config.

### Step 0: Working directory check

Check if the participant is in the right folder:

```bash
test -f .claude/settings.json && echo "OK" || echo "NOT_FOUND"
```

**If OK:** Proceed silently.

**If NOT_FOUND:** Check if the expected folder exists:
```bash
test -f ~/Projects/masterclass/.claude/settings.json && echo "EXISTS" || echo "MISSING"
```

- If EXISTS: "It looks like you're not in your project folder. Your project folder is at `~/Projects/masterclass/`. Please open it in VS Code (File > Open Folder) and start a new Claude Code session from there."
- If MISSING: "I can't find your project folder. It should have been created by the VS Code extension. Please go back to the extension setup panel and click **Create** next to Project Folder, then open that folder in VS Code."

In both cases, ask: "Would you like to continue anyway, or fix this first?"

If they continue, fire telemetry:
```bash
bash ${CLAUDE_PLUGIN_ROOT}/telemetry/send-event.sh orientation:wrong-directory "{\"cwd\":\"$(pwd)\",\"expectedPath\":\"~/Projects/masterclass\",\"settingsFound\":false}"
```

**Never blocking.** The check warns but does not prevent orientation from proceeding.

### Step 1: Check the telemetry config file

Run this command silently (do NOT show the raw output to the participant):

```bash
cat ~/.lah/telemetry-config.json 2>/dev/null
```

### Step 2: Handle the result

**If the file exists and has a non-empty `inviteCode`:**
→ Silently note the invite code. Tell the participant: "Connected to the workshop — your progress will be visible to instructors."
→ Proceed to Phase 0b.

**If the file exists but `inviteCode` is empty or missing:**
→ Ask: "I need your workshop invite code to connect you to the session. You can find it in the invite email you received. What's the code?"
→ Once they provide it, update the config file:

```bash
python3 -c "
import json, os
config_path = os.path.expanduser('~/.lah/telemetry-config.json')
with open(config_path) as f:
    config = json.load(f)
config['inviteCode'] = '$INVITE_CODE'
with open(config_path, 'w') as f:
    json.dump(config, f, indent=2)
print('Updated')
"
```

(Replace `$INVITE_CODE` with the actual code the participant provided.)

→ Then verify the connection (Step 3).

**If the file does not exist:**
→ Tell the participant: "It looks like the VS Code extension hasn't finished setting up yet. Let me create the telemetry config so your progress is tracked."
→ Ask for their invite code: "What's your workshop invite code? It's in your invite email."
→ Create the config:

```bash
mkdir -p ~/.lah && python3 -c "
import json, os
config = {
    'endpoint': 'https://adventurous-cricket-781.convex.site/plugin-telemetry',
    'token': '',
    'inviteCode': '$INVITE_CODE',
    'extensionVersion': 'plugin-bootstrap'
}
config_path = os.path.expanduser('~/.lah/telemetry-config.json')
with open(config_path, 'w') as f:
    json.dump(config, f, indent=2)
print('Created')
"
```

**Note:** Without a token, telemetry POSTs will return 401. This is acceptable — the participant can still complete orientation. Tell them: "I've saved your invite code. Telemetry may not sync until the VS Code extension runs its setup. You can continue."

### Step 3: Verify connection (only if token is present)

Send a lightweight verification ping to confirm the invite code works:

```bash
CONFIG=$(cat ~/.lah/telemetry-config.json 2>/dev/null)
ENDPOINT=$(echo "$CONFIG" | python3 -c "import json,sys; print(json.load(sys.stdin).get('endpoint',''))" 2>/dev/null)
TOKEN=$(echo "$CONFIG" | python3 -c "import json,sys; print(json.load(sys.stdin).get('token',''))" 2>/dev/null)
INVITE=$(echo "$CONFIG" | python3 -c "import json,sys; print(json.load(sys.stdin).get('inviteCode',''))" 2>/dev/null)

[ -z "$TOKEN" ] && echo "NO_TOKEN" && exit 0

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" -X POST "$ENDPOINT" \
  -H "Content-Type: application/json" \
  -H "x-lah-token: $TOKEN" \
  -d "{\"inviteCode\":\"$INVITE\",\"pluginName\":\"lah-orientation\",\"event\":\"orientation:config-verified\",\"payload\":{},\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}")

echo "$HTTP_CODE"
```

**If 200:** "Connected! Your progress will be visible to your instructors."
**If 401:** "Token issue — telemetry won't sync until the VS Code extension completes setup. You can still continue."
**If anything else:** "Couldn't reach the server — this might be a network issue. We'll continue anyway."

In all cases, **proceed with the orientation.** The config check is a best-effort step, never a blocker.

---

## Phase 0b: Check for Fast-Track or Pre-filled Context

Check `$ARGUMENTS` for input.

**If `$ARGUMENTS` contains explicit skip language** ("skip", "just get me started", "put me anywhere"):
→ Jump to **Fast-Track Path** (below)

**If `$ARGUMENTS` contains descriptive context** (e.g., "I'm a designer who wants to build a portfolio"):
→ Extract what you can (role, project description, etc.)
→ Skip questions that are already answered
→ Ask only what's missing
→ Continue with the appropriate round

**If `$ARGUMENTS` is empty:**
→ Start with Round 1

---

## Phase 1: Survey — Round 1 (About You)

Start with a brief confirmation and warm transition:

> "LAH Orientation loaded — let's get started. Now it's just you and me. I'll ask you a few questions to figure out the best track for you."

Then ask the Round 1 questions from `survey-questions.md` one at a time in a conversational flow:

1. **Name** — "What's your name?"
2. **Role** — "What's your role? For example: founder, developer, designer, student..."
3. **Coding experience** — "How would you rate your coding experience?" Share the options naturally.
4. **Terminal comfort** — "How comfortable are you with the terminal?" Share the options naturally.

**After Round 1:** Acknowledge their answers warmly. Summarize what you heard in 1-2 sentences. Then transition to Round 2.

Example: "Got it — you're a designer who's new to coding and hasn't used the terminal before. No worries at all, that's exactly what this workshop is designed for. Now let's talk about what you want to build."

---

## Phase 2: Survey — Round 2 (Your Project)

Ask the Round 2 questions from `survey-questions.md`:

1. **Has app idea** — "Do you have an idea for what you want to build today?"

**Branching based on answer:**

- **"clear" or "rough":** Ask for their build description: "Describe what you want to build in 1-2 sentences."
- **"multiple":** "Which one is the smallest and most self-contained? That's your best bet for a 4-hour build." Then ask for the description.
- **"no" (not yet):** **DO NOT fast-track.** Run the discovery conversation:
  - "What kind of work do you do day-to-day?"
  - "What's something that frustrates you or takes too long?"
  - "If you could have any tool or app built for you, what would it do?"
  - Use their answers to suggest 2-3 concrete, simple project ideas scoped for 4 hours.
  - Let them pick one. Then capture that as `buildDescription`.

2. **Has started building** — "Have you already started building this?"
3. **(If yes)** — "What exists so far? A repo, some code, a prototype?"

**After Round 2:** Brief acknowledgment, transition to Round 3.

---

## Phase 3: Survey — Round 3 (Your Goal)

Ask the final question from `survey-questions.md`:

1. **Primary goal** — "What's your main goal for today?" Share the options:
   - Build and launch an MVP
   - Learn to build with AI
   - Improve an existing project
   - Prototype and explore ideas
   - Automate a workflow

---

## Phase 4: Confirmation Gate

Present a clean summary of everything collected:

```
Here's what I've got:

- Name: [name]
- Role: [role]
- Coding experience: [codingExperience label]
- Terminal comfort: [terminalComfort label]
- Project idea: [buildDescription]
- Started building: [hasStartedBuilding]
- Main goal: [primaryGoal label]

Does this look right?
```

**DO NOT proceed until the participant explicitly confirms.** If they want to change something, update it and re-confirm.

---

## Phase 5: Routing

After confirmation, apply the routing rules from `${CLAUDE_PLUGIN_ROOT}/skills/orientation/references/routing-matrix.md`.

### Track classification

1. If `primaryGoal` is "workflow" → **assistant** track (confidence: 0.85)
2. Otherwise, scan `buildDescription` (lowercase) for keywords:
   - **ASSISTANT keywords:** automate, automation, workflow, pipeline, report, schedule, recurring, email sequence, scrape, monitor, digest, summarize daily, weekly report
   - **DEV keywords:** app, platform, website, tool, dashboard, landing page, saas, marketplace
3. Only assistant keywords → **assistant** (0.9)
4. Only dev keywords → **dev** (0.9)
5. Both → **dev** (0.4)
6. Neither → **dev** (0.0)

### Level assignment

- (`codingExperience` is none or beginner) AND (`terminalComfort` is never or scared) → **fundamental**
- (`codingExperience` is advanced or expert) AND (`terminalComfort` is comfortable or advanced) → **advanced**
- Everything else → **intermediate**

### Present the result

Follow the confidence-based presentation from `routing-matrix.md`:

- **High confidence (>= 0.8):** State the assignment directly.
- **Medium confidence (0.4 - 0.79):** State it but invite them to change if it doesn't feel right.
- **Low confidence (< 0.4):** State it as a starting point and explain it can be switched.

Briefly explain what their track and level mean (1-2 sentences each, from `routing-matrix.md`).

---

## Phase 6: Telemetry — Survey Completed

**Immediately after routing** (before report generation), fire the survey telemetry event in the background:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/telemetry/send-event.sh orientation:survey-completed "$(cat <<'PAYLOAD'
{
  "source": "plugin",
  "name": "[name]",
  "role": "[role]",
  "codingExperience": "[codingExperience]",
  "terminalComfort": "[terminalComfort]",
  "primaryGoal": "[primaryGoal]",
  "hasAppIdea": "[hasAppIdea]",
  "buildDescription": "[buildDescription]",
  "hasStartedBuilding": "[hasStartedBuilding]",
  "existingProjectDescription": "[existingProjectDescription or null]",
  "assignedTrack": "[track]",
  "assignedFlowLevel": "[level]",
  "trackConfidence": [confidence],
  "fastTracked": false
}
PAYLOAD
)"
```

Replace all `[placeholders]` with actual values. This event creates the backend survey record even if report generation fails.

---

## Phase 7: Report Generation

Dispatch the `orientation-advisor` agent with the participant profile:

```
Generate an orientation report for this participant:

PARTICIPANT PROFILE:
- Name: [name]
- Role: [role]
- Coding experience: [codingExperience]
- Terminal comfort: [terminalComfort]
- Primary goal: [primaryGoal]
- Assigned track: [track] / [level]

PROJECT IDEA:
- Has idea: [hasAppIdea]
- Description: [buildDescription]
- Started building: [hasStartedBuilding], [existingProjectDescription]

Assess this project for a [level]-level participant in a 4-hour workshop build session. Return your assessment as JSON.
```

Wait for the agent to return the JSON report.

---

## Phase 8: Report Display

Parse the advisor's JSON report and present it conversationally:

### Track & Level
"You're on the **[Track] track ([Level] level)**."

### Summary
Display the `summary` field.

### What to Build (MVP Scope)
Present `mvpScope` as a checklist:
- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3

### Stretch Goals
"If you finish early, you could also add:"
- Feature from `stretchGoals`

### Tech Stack
"You'll be using: **[techStack items joined]**"

### Challenges to Watch For
Present `challenges` as a brief list.

### Recommendation
Display the `recommendation` field.

### What's Next

Based on their level and track, tell them what's next. This is the last thing they see before leaving orientation — it must point them to the right place.

**If assigned Fundamental level (any track):**
> "You're oriented and ready to go! Your next step is **Module 1: Guided Build** — you'll pick an idea, plan it, and build it with Claude Code. It's your first experience of the full dev flow.
>
> After the guided build, you'll continue with the modules for your track. That's when you'll install your track plugin. You'll follow the same plugin installation steps you just learned."

**If assigned Intermediate or Advanced level (dev track):**
> "You're oriented and ready to go! You can choose to do **Module 1: Guided Build** if you want the practice first, or go straight to your track's first module:
>
> - **Module 2b: Research** — competitive analysis, design direction, architecture
>
> When you arrive at your first track module, it will tell you which plugin to install. You'll follow the same plugin installation steps you just learned."

**If assigned AI Assistant track (intermediate or advanced):**
> "You're oriented and ready to go! You can choose to do **Module 1: Guided Build** if you want the practice first, or go straight to:
>
> - **Module 2c: PDD** — Prompt, Define, Develop
>
> When you arrive at your first track module, it will tell you which plugin to install. You'll follow the same plugin installation steps you just learned."

---

## Phase 9: Telemetry — Report Generated

After displaying the report, fire the report telemetry event in the background:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/telemetry/send-event.sh orientation:report-generated "$(cat <<'PAYLOAD'
{
  "source": "plugin",
  "feasibility": "[feasibility]",
  "summary": "[summary]",
  "mvpScope": [mvpScope as JSON array],
  "stretchGoals": [stretchGoals as JSON array],
  "techStack": [techStack as JSON array],
  "challenges": [challenges as JSON array],
  "recommendation": "[recommendation]",
  "assignedTrack": "[track]",
  "assignedFlowLevel": "[level]",
  "buildDescription": "[buildDescription]"
}
PAYLOAD
)"
```

Done. The participant is oriented and ready to start building.

---

## Fast-Track Path

Only triggered by explicit skip language in `$ARGUMENTS`: "skip", "just get me started", "put me anywhere".

1. Assign defaults: **dev** track, **fundamental** level, confidence 0.0
2. Generate a generic starter report via the advisor agent with:
   - Role: "unknown"
   - Coding experience: "beginner"
   - Terminal comfort: "basic"
   - Primary goal: "learn"
   - Build description: "To be decided — participant chose to skip orientation and start building immediately"
3. Display the report (same format as Phase 8)
4. Fire the fast-track telemetry event:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/telemetry/send-event.sh orientation:fast-tracked "$(cat <<'PAYLOAD'
{
  "source": "plugin",
  "reason": "explicit-skip",
  "assignedTrack": "dev",
  "assignedFlowLevel": "fundamental",
  "trackConfidence": 0.0
}
PAYLOAD
)"
```

5. Tell them: "You're on the **Developer track (Fundamental level)** — the guided path. Your next step is **Module 1: Guided Build** — you'll pick an idea, plan it, and build it with Claude Code. You can always re-run `/orientation` later if you want a more tailored experience."
