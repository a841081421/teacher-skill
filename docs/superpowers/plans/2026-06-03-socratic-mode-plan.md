# Socratic Mode Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Socratic dialogue teaching mode as a new skill, with routing and preference persistence.

**Architecture:** New `socratic-core` skill handles Socratic-style teaching. The `learn` router reads `Teaching Mode` from `_progress.md` Profile to decide which teaching skill to invoke. `curriculum-builder` adds the field (default: Not Set). `teacher-core` gets a guard check. No changes to `note-generator` or `progress-tracker`.

**Tech Stack:** Superpowers skill framework (SKILL.md + YAML frontmatter).

---

## File Structure

```
skills/
  socratic-core/SKILL.md          ← NEW
  learn/SKILL.md                   ← MODIFY (routing)
  curriculum-builder/SKILL.md      ← MODIFY (Profile field)
  teacher-core/SKILL.md            ← MODIFY (startup guard)
  note-generator/SKILL.md          ← NO CHANGE
  progress-tracker/SKILL.md        ← NO CHANGE
```

---

### Task 1: Create socratic-core skill

**Files:**
- Create: `skills/socratic-core/SKILL.md`

- [ ] **Step 1: Write socratic-core/SKILL.md**

Create file at `D:/my-todo/teacher-skill/skills/socratic-core/SKILL.md` with this content:

```markdown
---
name: socratic-core
description: Use when the user chooses Socratic dialogue mode for learning — the teacher asks guiding questions instead of explaining directly
---

# Socratic Core

## Overview

Socratic dialogue teaching engine. Never explain — only ask. Let the student discover concepts through their own reasoning.

## When to Use

- User chose "Socratic Dialogue" mode for this session
- `_progress.md` Profile Teaching Mode is set to "Socratic"
- User explicitly says "苏格拉底模式" / "Socratic mode"

## Shared Logic

This skill shares startup and progress-update logic with `teacher-core`:
- **Startup**: Follow teacher-core's Startup process (read `_progress.md`, find current topic, check Log for "next" entry, announce today's topic)
- **Updating Progress**: Follow teacher-core's Updating Progress process (append to Log, mark topics in Plan)

Only the teaching flow and style differ below.

## Iron Law

```
NEVER DIRECTLY STATE THE ANSWER IN SOCRATIC MODE
ALWAYS LET THE STUDENT DISCOVER THE CONCEPT THROUGH THEIR OWN REASONING
```

## Teaching Flow

```
1. Open with a scenario or question related to the topic (do NOT explain the concept)
2. WAIT for the student to reason and respond
3. Evaluate the response:
   - Correct reasoning → acknowledge, ask a deeper follow-up question
   - Partially correct → ask a question that reveals the missing piece
   - Contains contradiction → ask a question that exposes the contradiction
   - Completely stuck → ask: "想继续想还是我给个提示？/ Keep thinking or get a hint?"
     - Keep thinking → wait, do not prompt again
     - Give hint → provide ONE clue, then continue questioning
4. After student discovers the core concept → help them summarize what THEY found
5. Move to next sub-topic with a new opening question
```

## Question Types

| Situation | Question Type | Example |
|-----------|---------------|---------|
| Opening new topic | Clarifying | "你觉得 X 是怎么工作的？/ What do you think X does?" |
| Answer too shallow | Probing | "为什么你这么认为？/ Why do you think that?" |
| Student contradicts self | Challenging | "但你刚才说 Y，这和 X 怎么共存？/ But you said Y earlier — how does that fit?" |
| Student stuck | Scaffolded hint | "想想 [相关知识]，它怎么用到这里？/ Think about [related], how might it apply?" |
| Going deeper | Extension | "如果 X 成立，改变 Y 会怎样？/ If X is true, what happens if we change Y?" |

## Teaching Style

- **Language**: Match the user's input language
- **Max output**: 200 words per turn (shorter than guided mode — questions should be concise)
- **Tone**: Curious, encouraging, never condescending
- **Forbidden**: Explaining concepts, giving definitions, lecturing

## Stuck Handling Protocol

When the student cannot answer after their attempt:

```
Ask: "你想继续想想，还是我给个提示？(Keep thinking or get a hint?)"

If "继续想" / "keep thinking":
  → Wait silently. Do not add hints or prompts.

If "提示" / "hint":
  → Give ONE concrete clue (a related concept, a partial analogy, a narrowed scope)
  → Then ask a new question based on that clue
  → If still stuck after 2 hints → ask again: keep thinking or switch to Guided mode?
```

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "This concept is too abstract, explaining directly is faster" | Direct explanation = Guided mode; Socratic's value is self-discovery |
| "Student tried 3 times, I should just tell them" | Student chose Socratic — respect the choice, give hints not answers |
| "I need to explain background before I can ask questions" | Turn background into a question: "你觉得在 X 出现之前，人们是怎么解决这个问题的？" |

## Red Flags

- You explained a concept directly → stop, rephrase as a question
- You gave the answer after the student said "keep thinking" → respect their choice
- You skipped the stuck-handling and just gave a hint → always ask first
- You are lecturing instead of asking → this is Guided mode behavior, switch or rephrase
- Your question can be answered with yes/no → rewrite as an open question
```

- [ ] **Step 2: Verify file content**

Run: `head -5 D:/my-todo/teacher-skill/skills/socratic-core/SKILL.md`
Expected: YAML frontmatter with `name: socratic-core` and description starting with "Use when"

Run: `grep -c "##" D:/my-todo/teacher-skill/skills/socratic-core/SKILL.md`
Expected: ~10 section headers

- [ ] **Step 3: Commit**

```bash
cd D:/my-todo/teacher-skill
git add skills/socratic-core/SKILL.md
git commit -m "feat: add socratic-core teaching skill"
```

---

### Task 2: Update curriculum-builder — add Teaching Mode to Profile

**Files:**
- Modify: `skills/curriculum-builder/SKILL.md`

- [ ] **Step 1: Edit the _progress.md template in curriculum-builder**

In `D:/my-todo/teacher-skill/skills/curriculum-builder/SKILL.md`, find the line that describes the Profile section and add the Teaching Mode field.

The current text says:
```
- **Profile**: goal, notes directory path, daily study time, total duration, created date
```

Change to:
```
- **Profile**: goal, notes directory path, daily study time, total duration, teaching mode (default: Not Set), created date
```

- [ ] **Step 2: Verify the edit**

Run: `grep "teaching mode" D:/my-todo/teacher-skill/skills/curriculum-builder/SKILL.md`
Expected: one match showing the updated Profile description

- [ ] **Step 3: Commit**

```bash
cd D:/my-todo/teacher-skill
git add skills/curriculum-builder/SKILL.md
git commit -m "feat: add Teaching Mode field to curriculum-builder Profile"
```

---

### Task 3: Update teacher-core — add mode guard to Startup

**Files:**
- Modify: `skills/teacher-core/SKILL.md`

- [ ] **Step 1: Add guard check to Startup section**

In `D:/my-todo/teacher-skill/skills/teacher-core/SKILL.md`, after the existing Startup step 4, add step 5.

Current Startup section:
```markdown
## Startup

1. Read `_progress.md` from the user's notes directory
2. Find the current topic from the Plan section (first unchecked item)
3. Check the Log section for "next continue" entry
4. Announce today's topic with a one-sentence preview
```

Add a 5th step:
```markdown
5. Check Profile Teaching Mode — if set to "Socratic", inform user:
   > "Your preferred mode is Socratic. Shall I use Socratic dialogue today, or switch to Guided Explanation?"
   If Socratic chosen → stop and suggest invoking socratic-core instead
```

- [ ] **Step 2: Update the description in frontmatter**

Current:
```yaml
description: Use when the user wants to start or continue a daily learning session, after _progress.md exists and has unfinished topics
```

Change to:
```yaml
description: Use when the user wants to start or continue a daily learning session in Guided Explanation mode, after _progress.md exists and has unfinished topics
```

- [ ] **Step 3: Verify edits**

Run: `grep "Socratic" D:/my-todo/teacher-skill/skills/teacher-core/SKILL.md`
Expected: 2 matches (frontmatter description + Startup step 5)

- [ ] **Step 4: Commit**

```bash
cd D:/my-todo/teacher-skill
git add skills/teacher-core/SKILL.md
git commit -m "feat: add Teaching Mode guard to teacher-core startup"
```

---

### Task 4: Update learn — add socratic-core routing

**Files:**
- Modify: `skills/learn/SKILL.md`

- [ ] **Step 1: Update the Routing Logic diagram**

In `D:/my-todo/teacher-skill/skills/learn/SKILL.md`, replace the Routing Logic section with this updated version that includes the Teaching Mode decision:

```markdown
## Routing Logic

```
User invokes /learn
       ↓
  _progress.md path known from prior session?
    Yes → Read _progress.md from notes directory
    No  → Ask: Where are your notes stored?
           (Skip this question for first-time users — curriculum-builder handles it)
       ↓
  Read _progress.md from notes directory
       ↓
  ┌──────────────────────────────────────┐
  │ File does not exist?                 │
  │   → Invoke curriculum-builder        │
  │                                      │
  │ File exists:                         │
  │   User says "study"/"continue"?      │
  │     Check Profile Teaching Mode:     │
  │       "Socratic" → socratic-core     │
  │       "Guided"   → teacher-core      │
  │       Not set → Ask user to choose,  │
  │                  save to Profile     │
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
  │                                      │
  │   User says "Socratic"/"苏格拉底"?    │
  │     → Save "Socratic" to Profile     │
  │     → Invoke socratic-core           │
  │                                      │
  │   User says "Guided"/"讲授"?         │
  │     → Save "Guided" to Profile       │
  │     → Invoke teacher-core            │
  └──────────────────────────────────────┘
```
```

- [ ] **Step 2: Update the Quick Reference table**

Replace the Quick Reference table with this updated version:

```markdown
## Quick Reference

| Condition | Route to |
|-----------|----------|
| `_progress.md` not found | `curriculum-builder` |
| "start", "study", "continue" / "开始学习", "继续" | Check Profile Teaching Mode → `teacher-core` or `socratic-core` |
| "done", "wrap up", "summarize" / "学完了", "总结", "结束" | `note-generator` |
| "progress", "plan", "adjust" / "进度", "调整计划", "跳过" | `progress-tracker` |
| User states a new goal | Confirm → `curriculum-builder` |
| "Socratic", "苏格拉底模式" | Save to Profile → `socratic-core` |
| "Guided", "讲授模式" | Save to Profile → `teacher-core` |
| Unclear intent | Ask: "Do you want to start learning, continue studying, check progress, or wrap up?" |
```

- [ ] **Step 3: Verify edits**

Run: `grep "socratic-core" D:/my-todo/teacher-skill/skills/learn/SKILL.md`
Expected: 3+ matches (Routing Logic diagram + Quick Reference table)

- [ ] **Step 4: Commit**

```bash
cd D:/my-todo/teacher-skill
git add skills/learn/SKILL.md
git commit -m "feat: add socratic-core routing to learn entry skill"
```

---

### Task 5: Sync marketplace copy and push

**Files:**
- Modify: `plugins/teacher-skill/skills/` (marketplace copy)

- [ ] **Step 1: Sync all skills to marketplace directory**

```bash
cp D:/my-todo/teacher-skill/skills/learn/SKILL.md D:/my-todo/teacher-skill/plugins/teacher-skill/skills/learn/SKILL.md
cp D:/my-todo/teacher-skill/skills/curriculum-builder/SKILL.md D:/my-todo/teacher-skill/plugins/teacher-skill/skills/curriculum-builder/SKILL.md
cp D:/my-todo/teacher-skill/skills/teacher-core/SKILL.md D:/my-todo/teacher-skill/plugins/teacher-skill/skills/teacher-core/SKILL.md
```

Create the socratic-core directory in marketplace:
```bash
mkdir -p D:/my-todo/teacher-skill/plugins/teacher-skill/skills/socratic-core
cp D:/my-todo/teacher-skill/skills/socratic-core/SKILL.md D:/my-todo/teacher-skill/plugins/teacher-skill/skills/socratic-core/SKILL.md
```

- [ ] **Step 2: Verify marketplace has 6 skill directories**

Run: `ls D:/my-todo/teacher-skill/plugins/teacher-skill/skills/`
Expected: `curriculum-builder  learn  note-generator  progress-tracker  socratic-core  teacher-core`

- [ ] **Step 3: Commit and push**

```bash
cd D:/my-todo/teacher-skill
git add -A
git commit -m "feat: add Socratic mode — new skill + routing + profile field"
git push origin master
```

---

### Task 6: Update local plugin installation

- [ ] **Step 1: Update the locally installed plugin**

```bash
claude plugin update teacher-skill-marketplace
```

Run: `claude plugin list`
Expected: `teacher-skill@teacher-skill-marketplace` shows updated version, status enabled

- [ ] **Step 2: Verify socratic-core is loaded**

Run: `ls C:/Users/84108/.claude/plugins/cache/teacher-skill-marketplace/teacher-skill/*/skills/socratic-core/SKILL.md`
Expected: file exists
