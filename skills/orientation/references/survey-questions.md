# Survey Questions — Orientation Plugin

Canonical question bank for the `/orientation` command. Field names match the `surveyResponses` schema exactly. Each question includes the conversational prompt, valid options, and a rephrasing guide for when participants say "I don't know."

---

## Round 1: About You

### `name`

**Ask:** "What's your name?"
**Type:** Free text
**Required:** Yes

---

### `role`

**Ask:** "What's your role? For example: founder, developer, designer, student..."
**Type:** Single select
**Options:**

| Value | Label | Description |
|-------|-------|-------------|
| `founder` | Founder | Running or starting a company |
| `developer` | Developer | Writing code professionally or as a hobby |
| `designer` | Designer | UI/UX, graphic, or product design |
| `product` | Product Manager | Managing product roadmaps and features |
| `marketing` | Marketing / Growth | Marketing, growth, content |
| `student` | Student | Currently studying |
| `other` | Other | None of the above |

**If "I don't know":** Rephrase: "What do you spend most of your day doing? That usually points to your role." Give a brief example for each option.

---

### `codingExperience`

**Ask:** "How would you rate your coding experience?"
**Type:** Single select
**Options:**

| Value | Label | Examples to share |
|-------|-------|-------------------|
| `none` | None | "I've never written code — not even HTML" |
| `beginner` | Beginner | "I've done a tutorial or two, maybe edited some HTML/CSS" |
| `intermediate` | Intermediate | "I can build simple things — a website, a script, small projects" |
| `advanced` | Advanced | "I build full apps, work with APIs, databases, deployment" |
| `expert` | Expert | "I write code professionally and could teach others" |

**If "I don't know":** Rephrase: "Have you ever written code before? Even HTML, a spreadsheet formula, or following a coding tutorial counts. If yes, you're at least beginner. If you've built something that works on your own, that's intermediate."

---

### `terminalComfort`

**Ask:** "How comfortable are you with the terminal?"
**Type:** Single select
**Options:**

| Value | Label | Examples to share |
|-------|-------|-------------------|
| `never` | Never used it | "I don't know what the terminal is" |
| `scared` | Intimidated by it | "I've seen it but it feels like hacker stuff" |
| `basic` | Know basic commands | "I can `cd`, `ls`, maybe `npm install`" |
| `comfortable` | Comfortable | "I use it regularly for git, running scripts, etc." |
| `advanced` | Power user | "I live in the terminal — tmux, vim, shell scripts" |

**If "I don't know":** Rephrase: "Have you ever opened Terminal on Mac or Command Prompt on Windows? Even once? If not, that's 'never used it' — and that's totally fine, we'll guide you through everything."

---

## Round 2: Your Project

### `hasAppIdea`

**Ask:** "Do you have an idea for what you want to build today?"
**Type:** Single select
**Options:**

| Value | Label | Next step |
|-------|-------|-----------|
| `clear` | Yes, I know exactly what I want | Ask `buildDescription` |
| `rough` | I have a rough idea | Ask `buildDescription` |
| `multiple` | I have multiple ideas | Ask them to pick the simplest: "Which one is the smallest and most self-contained? That's your best bet for a 4-hour build." Then ask `buildDescription` |
| `no` | Not yet | Trigger discovery conversation (see below) |

**Discovery conversation (when `hasAppIdea` is "no"):**

Do NOT fast-track. Instead, help them find an idea through 3 questions:
1. "What kind of work do you do day-to-day?"
2. "What's something that frustrates you or takes too long?"
3. "If you could have any tool or app built for you, what would it do?"

Use their answers to suggest 2-3 concrete starter ideas. Keep them simple and scoped for 4 hours. Let them pick one. Then fill in `buildDescription` based on their choice.

---

### `buildDescription`

**Ask:** "Describe what you want to build in 1-2 sentences."
**Type:** Free text (max 300 characters)
**Required:** Yes (after discovery if needed)

**Guidance:** Encourage specificity. "A dashboard" is too vague. "A personal finance dashboard that shows my monthly spending by category" is great.

---

### `hasStartedBuilding`

**Ask:** "Have you already started building this?"
**Type:** Single select
**Options:** `yes`, `no`

---

### `existingProjectDescription`

**Ask (only if `hasStartedBuilding` is "yes"):** "What exists so far? A repo, some code, a prototype?"
**Type:** Free text
**Required:** No (only asked conditionally)

---

## Round 3: Your Goal

### `primaryGoal`

**Ask:** "What's your main goal for today?"
**Type:** Single select
**Options:**

| Value | Label | Maps to |
|-------|-------|---------|
| `mvp` | Build and launch an MVP | Dev track signal |
| `learn` | Learn to build with AI | Neutral |
| `improve` | Improve an existing project | Dev track signal |
| `prototype` | Prototype and explore ideas | Neutral |
| `workflow` | Automate a workflow | **Strong** assistant track signal |

**If "I don't know":** Rephrase: "If the workshop goes perfectly, what do you walk away with? A working app you can show people? A new skill you didn't have before? A workflow that runs on autopilot?"

---

## Confirmation

After all rounds, present a summary of the collected answers:

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

**DO NOT proceed to routing until the participant explicitly confirms.**
