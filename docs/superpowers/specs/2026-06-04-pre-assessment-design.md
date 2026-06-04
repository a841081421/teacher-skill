# Pre-Assessment Design Spec

> Date: 2026-06-04
> Status: Awaiting user review

## Overview

Add an optional pre-assessment step during plan creation. The assessment evaluates the student's existing knowledge and marks weak areas in the learning plan, so teaching can focus on what the student actually needs to learn.

## Design Decisions

1. **Trigger**: Optional — asked after user confirms plan in curriculum-builder
2. **Result format**: Score + weak areas, all topics remain in plan (no deletion)
3. **Question format**: Hybrid — MCQ for basics (Familiar/Proficient), open-ended for core skills (Master)
4. **Implementation**: New `pre-assessment` skill

## Architecture

### New Skill: pre-assessment

**File:** `skills/pre-assessment/SKILL.md`

**YAML frontmatter:**
```yaml
---
name: pre-assessment
description: Use when the user wants to evaluate their existing knowledge before starting a learning plan, after confirming the skill tree
---
```

**Process:**

```
1. Read the confirmed skill tree / plan
2. Classify topics by question type:
   - Familiar/Proficient → 1 multiple-choice question each
   - Master → 1 open-ended question each
3. Ask questions one at a time, wait for user response
4. Evaluate responses:
   - MCQ correct → Solid
   - MCQ wrong → Blank
   - Open-ended: core logic correct → Solid
   - Open-ended: right direction, wrong details → Shaky
   - Open-ended: no idea / off-topic → Blank
5. Generate assessment report
6. Update _progress.md with assessment ratings
```

**MCQ Template (basic concepts):**

```
> 关于 [Topic]，以下哪个说法是正确的？
> A) [common misconception]
> B) [correct answer]
> C) [partially correct]
> D) [sounds right but wrong]
```

**Open-ended Template (core skills):**

```
> 用你自己的话解释 [Topic] 是怎么工作的。
> 如果你能用自己的话讲清楚，说明你真的理解了。
```

**Evaluation Criteria:**

| Question Type | Response | Rating |
|---------------|----------|--------|
| MCQ | Correct | Solid |
| MCQ | Wrong | Blank |
| Open-ended | Core logic correct, self-consistent | Solid |
| Open-ended | Right direction, wrong details | Shaky |
| Open-ended | No idea / off-topic | Blank |

**Topic Classification:**

| Required Proficiency | Question Type |
|---------------------|---------------|
| Familiar | MCQ (1 question) |
| Proficient | MCQ (1 question) |
| Master | Open-ended (1 question) |

**Assessment Report Format:**

```markdown
## Pre-Assessment Report

### Strong Areas
- [x] Topic A — Solid
- [x] Topic B — Solid

### Weak Areas
- [ ] Topic C — Shaky
- [ ] Topic D — Blank

### Suggested Focus
优先学习: Topic C, Topic D
```

**Iron Law:**
```
NEVER REMOVE TOPICS FROM THE PLAN BASED ON ASSESSMENT RESULTS
```

**Red Flags:**
- You deleted topics marked Solid → Solid means "understood", not "skip"
- You skipped a topic without asking → every topic in the plan must be assessed
- Assessment takes more than 1 question per topic → keep it to exactly 1

### Changes to Existing Skills

#### curriculum-builder/SKILL.md

After Step 3 (plan confirmed), add Step 3.5:

```markdown
### Step 3.5: Optional Pre-Assessment

Ask:
> "要不要先评估一下你已有的掌握情况？这样我们可以标注重点学习的领域。"

- Yes → invoke pre-assessment
- No → proceed to create _progress.md directly
```

#### teacher-core/SKILL.md

Startup step — add after announcing today's topic:

```markdown
If the topic has a Pre-assessment rating of Shaky or Blank:
  → add: "这个知识点之前评估为薄弱项，我们会重点讲解。"
```

#### socratic-core/SKILL.md

Shared Logic section — same as teacher-core: read Pre-assessment rating, mention it's a weak area when opening.

#### _progress.md Format Change

Plan section — topics can carry assessment ratings:

```markdown
## Plan
### Phase 1: [Name] (Week 1-2)
- [ ] Topic A — Master (Pre-assessment: Solid)
- [ ] Topic B — Proficient (Pre-assessment: Shaky)
- [ ] Topic C — Familiar
```

Topics without assessment have no parenthetical.

#### learn/SKILL.md

No changes.

#### note-generator/SKILL.md

No changes.

#### progress-tracker/SKILL.md

No changes.

## File Changes Summary

| File | Action | Scope |
|------|--------|-------|
| `skills/pre-assessment/SKILL.md` | Create | New skill |
| `skills/curriculum-builder/SKILL.md` | Edit | +1 optional step |
| `skills/teacher-core/SKILL.md` | Edit | Startup +1 line |
| `skills/socratic-core/SKILL.md` | Edit | Shared Logic +1 note |
| `skills/learn/SKILL.md` | None | — |
| `skills/note-generator/SKILL.md` | None | — |
| `skills/progress-tracker/SKILL.md` | None | — |
