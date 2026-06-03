# Socratic Mode Design Spec

> Date: 2026-06-03
> Status: Awaiting user review

## Overview

Add a Socratic dialogue teaching mode to the teacher-skill system. Instead of explaining first and verifying after, the Socratic mode asks guiding questions and lets the student discover concepts through their own reasoning.

## Design Decisions

1. **Mode selection**: User chooses each session (Guided Explanation or Socratic Dialogue)
2. **Stuck handling**: Ask student "want to keep thinking or get a hint?" — never force
3. **Preference persistence**: Save to `_progress.md` Profile, default to last used mode
4. **Implementation**: New `socratic-core` skill (not merged into teacher-core)

## Architecture

### New Skill: socratic-core

**File:** `skills/socratic-core/SKILL.md`

**YAML frontmatter:**
```yaml
---
name: socratic-core
description: Use when the user chooses Socratic dialogue mode for learning — the teacher asks guiding questions instead of explaining directly
---
```

**Responsibility split (cross-reference pattern):**

| Part | Owner |
|------|-------|
| Startup (read _progress.md, find topic) | Reference teacher-core Startup |
| Teaching flow (Socratic questioning) | socratic-core own |
| Stuck handling (student chooses) | socratic-core own |
| Updating Progress (write Log, mark complete) | Reference teacher-core Updating Progress |
| Teaching style rules | socratic-core own |
| Rationalization Table | socratic-core own |

**Teaching Flow:**

```
1. Open with a scenario/question (do NOT explain the concept itself)
2. Wait for student to reason and respond
3. Evaluate response:
   - Correct reasoning → acknowledge, ask deeper follow-up
   - Partially correct → ask a question revealing the gap
   - Contradiction → ask a question exposing the contradiction
   - Completely stuck → ask "want to keep thinking or get a hint?"
     - Keep thinking → wait
     - Give hint → provide one clue, continue questioning
4. Student discovers core concept → help summarize their finding
5. Move to next sub-topic with a new opening question
```

**Question Types (Quick Reference):**

| Situation | Question Type | Example |
|-----------|---------------|---------|
| Opening new topic | Clarifying | "What do you think X means?" |
| Shallow answer | Probing | "Why do you think that?" |
| Student contradicts self | Challenging | "But you said Y earlier — how does that fit?" |
| Student stuck | Scaffolded hint | "Think about [related concept]. How might it apply here?" |
| Going deeper | Extension | "If X is true, what happens if we change Y?" |

**Iron Law:**

```
NEVER DIRECTLY STATE THE ANSWER IN SOCRATIC MODE
ALWAYS LET THE STUDENT DISCOVER THE CONCEPT THROUGH THEIR OWN REASONING
```

**Rationalization Table:**

| Excuse | Reality |
|--------|---------|
| "This concept is too abstract, explaining directly is faster" | Direct explanation = Guided mode; Socratic's value is self-discovery |
| "Student tried 3 times, I should just tell them" | Student chose Socratic mode — respect the choice, give hints not answers |
| "I need to explain background before I can ask questions" | Turn background into a question: "How do you think people solved this before X existed?" |

**Red Flags:**

- You explained a concept directly → stop, rephrase as a question
- You gave the answer after the student said "keep thinking" → respect their choice
- You skipped the stuck-handling step and just gave a hint → always ask first
- You are lecturing instead of questioning → same red flag as teacher-core

### Changes to Existing Skills

#### learn/SKILL.md

Quick Reference table — add one row:

| Condition | Route to |
|-----------|----------|
| `_progress.md` not found | `curriculum-builder` |
| "start", "study", "continue" / "开始学习", "继续" | Check Profile Teaching Mode → `teacher-core` or `socratic-core` |
| "done", "wrap up", "summarize" / "学完了", "总结", "结束" | `note-generator` |
| "progress", "plan", "adjust" / "进度", "调整计划", "跳过" | `progress-tracker` |
| User states a new goal | Confirm → `curriculum-builder` |
| Unclear intent | Ask |
| User explicitly says "Socratic mode" / "苏格拉底模式" | `socratic-core` |
| User explicitly says "Guided mode" / "讲授模式" | `teacher-core` |

Routing Logic — teaching mode decision:

```
User says "study"/"continue"
       ↓
  _progress.md Profile has Teaching Mode?
    "Socratic" → Invoke socratic-core
    "Guided"   → Invoke teacher-core
    Not set    → Ask: "Teaching style today?
                  (1) Guided Explanation (2) Socratic Dialogue"
                  Save choice to Profile
```

#### curriculum-builder/SKILL.md

_progress.md Profile — add one field:

```markdown
## Profile
- Goal: [user's goal]
- Notes Directory: [path]
- Daily Study Time: [X] hours
- Total Duration: [X] weeks
- Teaching Mode: Not Set
- Created: [YYYY-MM-DD]
```

No other changes. Mode is selected during first learning session, not during plan creation.

#### teacher-core/SKILL.md

Startup — add one check:

```
After reading _progress.md:
5. Check Profile Teaching Mode
   If "Socratic" → inform user and suggest using socratic-core instead
```

Add one line to description: this is the Guided Explanation teaching engine.

#### note-generator/SKILL.md

No changes. Both modes produce the same note format.

#### progress-tracker/SKILL.md

No changes. Progress management is mode-agnostic.

## File Changes Summary

| File | Action | Scope |
|------|--------|-------|
| `skills/socratic-core/SKILL.md` | Create | New skill |
| `skills/learn/SKILL.md` | Edit | Routing table + routing logic |
| `skills/curriculum-builder/SKILL.md` | Edit | Profile template +1 field |
| `skills/teacher-core/SKILL.md` | Edit | Startup +1 check, description update |
| `skills/note-generator/SKILL.md` | None | — |
| `skills/progress-tracker/SKILL.md` | None | — |
