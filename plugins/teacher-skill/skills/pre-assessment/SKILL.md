---
name: pre-assessment
description: Use when the user wants to evaluate their existing knowledge before starting a learning plan, after confirming the skill tree
---

# Pre-Assessment

## Overview

Evaluate the student's existing knowledge before learning begins. Identify strong and weak areas so teaching can focus on what matters.

## When to Use

- After curriculum-builder confirms the plan
- User agrees to pre-assessment when asked
- User explicitly says "评估一下" / "test my knowledge"

## Iron Law

```
NEVER REMOVE TOPICS FROM THE PLAN BASED ON ASSESSMENT RESULTS
```

## Process

### Step 1: Read the Plan

Read the confirmed skill tree. Classify each topic by question type:

| Required Proficiency | Question Type |
|---------------------|---------------|
| Familiar | Multiple-choice (1 question) |
| Proficient | Multiple-choice (1 question) |
| Master | Open-ended (1 question) |

### Step 2: Ask Questions

Go through topics one at a time. For each topic, ask exactly 1 question.

**MCQ Template (Familiar/Proficient):**

```
> 关于 [Topic]，以下哪个说法是正确的？
> A) [common misconception]
> B) [correct answer]
> C) [partially correct but incomplete]
> D) [sounds right but wrong]
```

Rules for MCQ:
- Correct answer must be clearly correct, not a trick
- Each distractor should target a specific misunderstanding
- Shuffle answer order randomly (don't always put correct answer in same position)

**Open-ended Template (Master):**

```
> 用你自己的话解释 [Topic] 是怎么工作的。
> 如果你能用自己的话讲清楚，说明你真的理解了。
```

WAIT for the user's answer before moving to the next topic.

### Step 3: Evaluate Responses

| Question Type | Response | Rating |
|---------------|----------|--------|
| MCQ | Correct | Solid |
| MCQ | Wrong | Blank |
| Open-ended | Core logic correct, self-consistent | Solid |
| Open-ended | Right direction, wrong details | Shaky |
| Open-ended | No idea / off-topic | Blank |

Briefly tell the user the result after each answer:
- Solid → "掌握得不错！"
- Shaky → "方向对，但有些细节需要补充。"
- Blank → "没关系，这正是我们要学的。"

### Step 4: Generate Report

After all topics assessed, present the report:

```markdown
## Pre-Assessment Report

### Strong Areas
- [x] Topic A — Solid
- [x] Topic B — Solid

### Weak Areas
- [ ] Topic C — Shaky
- [ ] Topic D — Blank

### Suggested Focus
优先学习: [list Shaky and Blank topics]
```

### Step 5: Update Progress

Create `_progress.md` with assessment ratings in the Plan section:

```markdown
## Plan
### Phase 1: [Name] (Week 1-2)
- [ ] Topic A — Master (Pre-assessment: Solid)
- [ ] Topic B — Proficient (Pre-assessment: Shaky)
- [ ] Topic C — Familiar
```

Topics rated Solid keep their check box unchecked — they still appear in the plan for review.
Topics not assessed have no parenthetical.

## Red Flags

- You deleted topics marked Solid → Solid means "understood", not "skip"
- You asked more than 1 question per topic → keep it to exactly 1
- You skipped a topic → every topic in the plan must be assessed
- You made the assessment feel like an exam → keep it light and encouraging
