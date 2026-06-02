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

After confirmation, create `_progress.md` in the notes directory with three sections:
- **Profile**: goal, notes directory path, daily study time, total duration, created date
- **Plan**: phases from the confirmed plan, each topic as `- [ ] Topic — Level`
- **Log**: empty section (entries added by teacher-core and note-generator)

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The user seemed happy with the skill tree, I'll skip plan confirmation" | Happiness ≠ approval; always get explicit "yes" or "looks good" |
| "This is a simple goal, I can combine steps" | Simple goals still need structured plans; skipping steps creates misaligned expectations |
| "I'll just use a generic template" | Generic plans fail; every goal needs tailored skill decomposition |

## Red Flags

Stop immediately if:
- You are generating a plan before the user confirmed the skill tree
- The user said "whatever" or "you decide" — provide a recommendation but still require explicit confirmation
- You added topics the user didn't agree to
