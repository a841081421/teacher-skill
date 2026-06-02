# Teacher Skill 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现一套通用的 AI 学习助手 skill（5 个 skill），用户输入 `/learn` 即可完成目标拆解 → 计划制定 → 每日教学 → 总结归档的全流程。

**Architecture:** 5 个 skill 按职责拆分：`learn` 作为路由入口，`curriculum-builder` 负责目标拆解和计划制定，`teacher-core` 负责一问一答教学，`progress-tracker` 负责进度管理，`note-generator` 负责笔记生成。所有 skill 通过读写用户笔记目录下的 `_progress.md` 协调状态。

**Tech Stack:** Superpowers skill framework（SKILL.md + YAML frontmatter），支持 Claude Code 和 OpenCode。

---

## File Structure

```
teacher-skill/
  skills/
    learn/SKILL.md                  ← 入口路由
    curriculum-builder/SKILL.md     ← 目标拆解 + 计划制定
    teacher-core/SKILL.md           ← 一问一答教学
    progress-tracker/SKILL.md       ← 进度管理
    note-generator/SKILL.md         ← 笔记生成
  docs/
    superpowers/
      specs/2026-06-02-teacher-skill-design.md
      plans/2026-06-02-teacher-skill-plan.md
```

---

### Task 1: 项目初始化

**Files:**
- Create: `.gitignore`
- Modify: none

- [ ] **Step 1: 创建 .gitignore**

```
.DS_Store
*.swp
*.swo
*~
```

- [ ] **Step 2: 提交项目骨架**

Run: `cd D:/my-todo/teacher-skill && git add .gitignore docs/ && git commit -m "chore: init teacher-skill project with design spec"`

---

### Task 2: curriculum-builder skill

这是第一个要实现的 skill，因为它定义了 `_progress.md` 的格式——其他所有 skill 都依赖这个格式。

**Files:**
- Create: `skills/curriculum-builder/SKILL.md`

- [ ] **Step 1: 写 curriculum-builder/SKILL.md**

```markdown
---
name: curriculum-builder
description: Use when the user wants to start learning something new and needs a learning plan, or when _progress.md does not exist in the notes directory
---

# Curriculum Builder

## Overview

Take a user's learning goal, break it into a structured skill tree with proficiency levels, then generate a phased learning plan.

## When to Use

- User says they want to learn something (new goal, career target, exam prep)
- `_progress.md` does not exist in the user's notes directory
- User explicitly asks to create or redo a learning plan

## Iron Law

```
NEVER SKIP THE USER CONFIRMATION STEP FOR SKILL TREE AND PLAN
```

## Process

Execute these 3 steps **sequentially**, waiting for user confirmation after each.

### Step 1: Goal Decomposition

Ask the user:
> What is your learning goal? (e.g., "pass CET-6", "get a Java backend job", "learn RAG from scratch")

Based on the goal, produce a **Skill Tree** table:

```markdown
| Knowledge Area | Skill/Topic | Proficiency Required |
|----------------|-------------|---------------------|
| [Area 1] | [Topic 1.1] | Master / Proficient / Familiar |
| [Area 1] | [Topic 1.2] | Proficient |
| [Area 2] | [Topic 2.1] | Familiar |
```

**Proficiency levels:**
- **Master**: Can explain from first principles, implement from scratch, handle variants
- **Proficient**: Understand principles, use correctly, explain design choices
- **Familiar**: Know what it is, what problem it solves, general approach

Present the table and **wait for user to confirm or adjust** before proceeding.

### Step 2: Learning Parameters

Ask these questions (one at a time if needed):

1. > How many hours per day can you study? (e.g., 1, 2, 3 hours)
2. > How many weeks do you have in total? (e.g., 4, 8, 12 weeks)
3. > Where should I save your notes? (default: `./notes`)

### Step 3: Generate Phased Plan

Based on skill count + study hours, distribute topics into phases.

```markdown
## Learning Plan

### Phase 1: [Name] (Week 1-2)
- [ ] Topic A — Master
- [ ] Topic B — Proficient

### Phase 2: [Name] (Week 3-4)
- [ ] Topic C — Familiar

**Total:** X topics across Y phases, estimated Z hours
```

Present the plan and **wait for user confirmation**.

After confirmation, create `_progress.md` in the notes directory:

```markdown
# Learning Progress

## Profile
- Goal: [user's goal]
- Notes Directory: [path]
- Daily Study Time: [X] hours
- Total Duration: [X] weeks
- Created: [YYYY-MM-DD]

## Plan
### Phase 1: [Name] (Week 1-2)
- [ ] Topic A — Master
- [ ] Topic B — Proficient

### Phase 2: [Name] (Week 3-4)
- [ ] Topic C — Familiar

## Log
<!-- Entries added by teacher-core and note-generator -->
```

## Red Flags

Stop immediately if:
- You are generating a plan before the user confirmed the skill tree
- The user said "whatever" or "you decide" — provide a recommendation but still require explicit confirmation
- You added topics the user didn't agree to
```

- [ ] **Step 2: 验证文件内容**

Run: `cat D:/my-todo/teacher-skill/skills/curriculum-builder/SKILL.md`
Expected: 完整的 SKILL.md 内容，YAML frontmatter 格式正确

- [ ] **Step 3: 提交**

Run: `cd D:/my-todo/teacher-skill && git add skills/curriculum-builder/SKILL.md && git commit -m "feat: add curriculum-builder skill"`

---

### Task 3: teacher-core skill

**Files:**
- Create: `skills/teacher-core/SKILL.md`

- [ ] **Step 1: 写 teacher-core/SKILL.md**

```markdown
---
name: teacher-core
description: Use when the user wants to start or continue a daily learning session, after _progress.md exists and has unfinished topics
---

# Teacher Core

## Overview

One-concept-one-question teaching engine. Teach, verify, correct, repeat.

## When to Use

- User says "start learning", "continue", "let's study"
- `_progress.md` exists with unfinished topics

## Iron Law

```
NEVER MOVE TO THE NEXT TOPIC WITHOUT A VERIFIED UNDERSTANDING CHECK
NEVER DUMP A WALL OF TEXT — ONE CONCEPT, ONE QUESTION AT A TIME
```

## Startup

1. Read `_progress.md` from the user's notes directory
2. Find the current topic from the Plan section (first unchecked item)
3. Check the Log section for "next continue" entry
4. Announce today's topic with a one-sentence preview

## Teaching Flow

```
1. Introduce concept from intuition/analogy (NO jargon first)
2. Explain ONE sub-topic
3. Ask ONE verification question
4. WAIT for user's answer (do not continue until answered)
5. Evaluate:
   - Correct → acknowledge, move to next sub-topic
   - Wrong → gently correct, explain from a different angle, ask again
6. After all sub-topics covered → present ALL review questions at once
7. After user answers all → update _progress.md Log section
```

## Teaching Style

- **Language**: Match the user's input language
- **Tone**: Concise, direct, no filler
- **Explanation**: Analogy first, then details. Concrete numbers, not vague descriptions
- **Max output**: 300 words per turn. If you need more, split into another turn
- **Jargon**: Only introduce after the concept is understood through analogy

## Verification Questions

- Must test understanding, not memorization
- Prefer "explain why" and "what would happen if" over "what is X"
- At least 1 question per sub-topic
- Final review: all questions presented together, user answers all

## Updating Progress

After each session, append to `_progress.md` Log section:

```markdown
### YYYY-MM-DD
- Completed: [topic name] (full or partial)
- Notes: [filename if note was generated]
- Next: [what to continue next time]
- Mistakes corrected: [list any misunderstandings]
```

Mark topics as complete in the Plan section:
- `- [x] Topic A — Master` (fully understood)
- Leave as `- [ ]` if partially covered

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "This topic is too simple for verification" | Simple misconceptions are the hardest to detect; verification takes 10 seconds |
| "The user said they understand" | Saying "I get it" ≠ understanding; verify with a concrete question |
| "Teaching multiple topics at once is more efficient" | Information dump ≠ learning; Q&A reveals blind spots |
| "The user seems bored, I should speed up" | Speed without verification builds false confidence |

## Red Flags

Stop and recalibrate if:
- You output more than 300 words in a single turn
- You moved to a new sub-topic without asking a verification question
- You accepted "I understand" as a verified answer
- You are lecturing instead of asking
```

- [ ] **Step 2: 验证文件内容**

Run: `cat D:/my-todo/teacher-skill/skills/teacher-core/SKILL.md`
Expected: 完整内容，frontmatter 正确

- [ ] **Step 3: 提交**

Run: `cd D:/my-todo/teacher-skill && git add skills/teacher-core/SKILL.md && git commit -m "feat: add teacher-core skill"`

---

### Task 4: note-generator skill

**Files:**
- Create: `skills/note-generator/SKILL.md`

- [ ] **Step 1: 写 note-generator/SKILL.md**

```markdown
---
name: note-generator
description: Use when the user wants to end a study session, when the user says "done for today", "summarize", "wrap up", or "generate notes"
---

# Note Generator

## Overview

Convert a teaching session into a structured Obsidian-friendly Markdown note and update progress.

## When to Use

- User says "done for today", "let's wrap up", "summarize", "generate notes"
- End of a teaching session

## Iron Law

```
NEVER GENERATE NOTES WITHOUT INCLUDING CORRECTED MISTAKES
```

## Process

### Step 1: Review Session

Scan the conversation history for:
- All topics taught this session
- Key explanations and analogies used
- Questions the user answered correctly
- Mistakes the user made and corrections given

### Step 2: Generate Note

Use this template:

```markdown
# [Topic Name]

> Date: YYYY-MM-DD
> Phase: [current phase name from _progress.md]

## Key Concepts

### [Sub-topic 1]
- [Core points in plain language]
- Key formula / code (if any)

### [Sub-topic 2]
- [Core points]

## Corrected Mistakes
- **Misunderstanding:** [what the user got wrong]
  **Correction:** [the correct explanation]

## Review Questions
1. [Question from session] — Answer: [correct answer]

## Next Steps
- [ ] [Next topic to learn]
```

### Step 3: Save Note

1. Read the notes directory from `_progress.md` Profile section
2. Generate filename: `YYYY-MM-DD-[topic-kebab-case].md`
3. Confirm filename with user
4. Write the file

### Step 4: Update Progress

Append to `_progress.md` Log section:

```markdown
### YYYY-MM-DD
- Completed: [topic] (full/partial)
- Notes: [filename]
- Next: [what to continue]
- Mistakes corrected: [list]
```

Mark completed topics in Plan section: `- [x]`

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Writing notes in technical jargon | Use the same language level used during teaching |
| Omitting corrected mistakes | These are the most valuable part — always include |
| Making the note too long | Notes are for review, not re-learning. Keep each point to 1-2 sentences |
| Forgetting to update `_progress.md` | Progress file is the single source of truth; always update it |

## Red Flags

- Note has no "Corrected Mistakes" section → go back and add it
- Note is longer than the actual teaching session → trim to essentials
- `_progress.md` not updated after note generation → update it now
```

- [ ] **Step 2: 验证文件内容**

Run: `cat D:/my-todo/teacher-skill/skills/note-generator/SKILL.md`
Expected: 完整内容，frontmatter 正确

- [ ] **Step 3: 提交**

Run: `cd D:/my-todo/teacher-skill && git add skills/note-generator/SKILL.md && git commit -m "feat: add note-generator skill"`

---

### Task 5: progress-tracker skill

**Files:**
- Create: `skills/progress-tracker/SKILL.md`

- [ ] **Step 1: 写 progress-tracker/SKILL.md**

```markdown
---
name: progress-tracker
description: Use when the user wants to check learning progress, skip a topic, adjust the plan, or reports falling behind schedule
---

# Progress Tracker

## Overview

Manage and adapt the learning plan. Read progress, handle exceptions, adjust when life happens.

## When to Use

- User says "check progress", "how am I doing", "show my plan"
- User says "skip this", "this is too hard", "adjust plan"
- User says "I haven't studied in a while"
- User wants to change their goal mid-way

## Iron Law

```
NEVER DELETE COMPLETED LEARNING RECORDS WHEN ADJUSTING PLANS
```

## Quick Reference

| User says | Action |
|-----------|--------|
| "check progress" | Read `_progress.md`, report: completed X/Y topics, current phase, total study days |
| "skip this topic" | Mark as skipped with reason in Log, update "Next" pointer |
| "this is too hard" | Break current topic into 2-3 smaller sub-topics, lower expectation for today |
| "adjust plan" | Re-ask study hours/duration, redistribute phases, keep all completed records |
| "haven't studied in a while" | Gently acknowledge, suggest reviewing last session's notes, resume from "Next" pointer |
| "change my goal" | Confirm → rename `_progress.md` to `_progress_old.md` → invoke curriculum-builder |

## Progress Report Format

When showing progress, use this format:

```markdown
📊 Progress: Phase [N] — [Phase Name]

✅ Completed: X/Y topics ([percentage]%)
📅 Study days: [N] days
🎯 Current: [topic name]
⏭️ Next: [next topic name]
```

## Exception Handling

| Situation | Response |
|-----------|----------|
| `_progress.md` missing or corrupted | Tell user exactly what's wrong, offer to recreate via curriculum-builder or attempt manual fix |
| All topics completed | Congratulate! Suggest: review weak areas or set a new goal |
| `_progress_old.md` exists | Ask: resume old plan or continue with current one? |
| Gap > 7 days since last log entry | Gentle reminder, suggest reviewing last notes before new content |

## Plan Adjustment Rules

When redistributing phases:
1. Keep all `- [x]` completed items unchanged
2. Keep all Log entries unchanged
3. Only redistribute unchecked items
4. Confirm the new plan with user before saving
5. Update only the Plan section and Profile parameters

## Red Flags

- You deleted Log entries to "clean up" → they are learning history, never delete
- You adjusted plan without user confirmation → always confirm changes
- User sounds frustrated → acknowledge the difficulty, offer to simplify, don't push through
```

- [ ] **Step 2: 验证文件内容**

Run: `cat D:/my-todo/teacher-skill/skills/progress-tracker/SKILL.md`
Expected: 完整内容，frontmatter 正确

- [ ] **Step 3: 提交**

Run: `cd D:/my-todo/teacher-skill && git add skills/progress-tracker/SKILL.md && git commit -m "feat: add progress-tracker skill"`

---

### Task 6: learn 入口 skill

这个 skill 放最后写，因为它需要引用其他 4 个 skill 的名称和触发条件。

**Files:**
- Create: `skills/learn/SKILL.md`

- [ ] **Step 1: 写 learn/SKILL.md**

```markdown
---
name: learn
description: Use when the user types /learn or wants to start learning, continue studying, check progress, or wrap up a study session
---

# Learn

## Overview

Entry point for the teacher skill system. Read progress, route to the right sub-skill.

## When to Use

- User types `/learn`
- User says "I want to learn X", "let's study", "continue learning"

## Iron Law

```
NEVER START TEACHING WITHOUT CHECKING _progress.md FIRST
```

## Routing Logic

```
User invokes /learn
       ↓
  Ask: Where are your notes stored? (only if not remembered from prior session)
       ↓
  Read _progress.md from notes directory
       ↓
  ┌──────────────────────────────────────┐
  │ File does not exist?                 │
  │   → Invoke curriculum-builder        │
  │                                      │
  │ File exists:                         │
  │   User says "study"/"continue"?      │
  │     → Invoke teacher-core            │
  │                                      │
  │   User says "done"/"summarize"?      │
  │     → Invoke note-generator          │
  │                                      │
  │   User says "progress"/"adjust"?     │
  │     → Invoke progress-tracker        │
  │                                      │
  │   User states a new goal?            │
  │     → Confirm abandon old plan?      │
  │       Yes → Invoke curriculum-builder│
  │       No  → Continue current plan    │
  └──────────────────────────────────────┘
```

## Quick Reference

| Condition | Route to |
|-----------|----------|
| `_progress.md` not found | `curriculum-builder` |
| User says "start", "study", "continue" | `teacher-core` |
| User says "done", "wrap up", "summarize", "notes" | `note-generator` |
| User says "progress", "plan", "adjust", "skip" | `progress-tracker` |
| User states a new goal | Confirm → `curriculum-builder` |
| Unclear intent | Ask: "Do you want to start learning, continue studying, check progress, or wrap up?" |

## First-Time User

If this is the first interaction (no prior context):
1. Greet briefly
2. Ask: "What would you like to learn?"
3. Proceed to `curriculum-builder`

## Returning User

If `_progress.md` exists:
1. Show brief status: current phase, next topic
2. Ask what they want to do
3. Route accordingly

## Red Flags

- You started teaching directly without reading `_progress.md` → stop and read it first
- You don't know which sub-skill to route to → ask the user
- User's intent is ambiguous → clarify with a multiple-choice question, don't guess
```

- [ ] **Step 2: 验证文件内容**

Run: `cat D:/my-todo/teacher-skill/skills/learn/SKILL.md`
Expected: 完整内容，frontmatter 正确

- [ ] **Step 3: 提交**

Run: `cd D:/my-todo/teacher-skill && git add skills/learn/SKILL.md && git commit -m "feat: add learn entry skill"`

---

### Task 7: 最终验证 + 一次性提交

- [ ] **Step 1: 验证所有文件存在**

Run: `ls -R D:/my-todo/teacher-skill/skills/`
Expected: 5 个目录，每个包含 SKILL.md

- [ ] **Step 2: 验证所有 frontmatter 格式**

Run: `grep -r "^name:" D:/my-todo/teacher-skill/skills/*/SKILL.md`
Expected: 5 行，每行 `name:` 开头，值带连字符无空格

Run: `grep -r "^description:" D:/my-todo/teacher-skill/skills/*/SKILL.md`
Expected: 5 行，每行以 `Use when` 开头

- [ ] **Step 3: 最终提交**

Run: `cd D:/my-todo/teacher-skill && git add docs/superpowers/plans/2026-06-02-teacher-skill-plan.md && git commit -m "docs: add implementation plan"`
