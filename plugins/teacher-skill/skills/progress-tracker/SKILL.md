---
name: progress-tracker
description: Use when the user wants to check learning progress, skip a topic, adjust the plan, or reports falling behind schedule
---

# Progress Tracker

## Overview

Manage and adapt the learning plan. Read progress, handle exceptions, adjust when life happens.

## When to Use

- User says "check progress", "how am I doing" / "看看进度", "我学到哪了"
- User says "skip this", "this is too hard" / "跳过", "太难了", "调整计划"
- User says "I haven't studied in a while" / "好几天没学了"
- User wants to change their goal mid-way

## Iron Law

```
NEVER DELETE COMPLETED LEARNING RECORDS WHEN ADJUSTING PLANS
```

## Quick Reference

| User says | Action |
|-----------|--------|
| "check progress" / "看看进度" | Read `_progress.md`, report: completed X/Y topics, current phase, total study days |
| "skip this topic" / "跳过" | Mark as skipped with reason in Log, update "Next" pointer |
| "this is too hard" / "太难了" | Break current topic into 2-3 smaller sub-topics, lower expectation for today |
| "adjust plan" / "调整计划" | Re-ask study hours/duration, redistribute phases, keep all completed records |
| "haven't studied in a while" / "好久没学" | Gently acknowledge, suggest reviewing last session's notes, resume from "Next" pointer |
| "change my goal" / "换个目标" | Confirm → rename `_progress.md` to `_progress_old.md` → invoke curriculum-builder |

## Progress Report Format

When showing progress, use this format:

```markdown
Progress: Phase [N] — [Phase Name]

Completed: X/Y topics ([percentage]%)
Study days: [N] days
Current: [topic name]
Next: [next topic name]
```

## Exception Handling

| Situation | Response |
|-----------|----------|
| `_progress.md` missing or corrupted | Tell user exactly what's wrong, offer to recreate via curriculum-builder or attempt manual fix |
| All topics completed | Congratulate! Suggest: review weak areas or set a new goal |
| `_progress_old.md` exists | Ask: resume old plan or continue with current one? |
| Gap > 7 days since last log entry | Gentle reminder, suggest reviewing last notes before new content |

## Validation Checklist

When reading `_progress.md`, verify it has:
1. `## Profile` section with: Goal, Notes Directory, Daily Study Time, Total Duration, Created
2. `## Plan` section with at least one phase containing topics
3. `## Log` section (can be empty)

If any section is missing or malformed, treat as corrupted and follow the Exception Handling table above.

## Plan Adjustment Rules

When redistributing phases:
1. Keep all `- [x]` completed items unchanged
2. Keep all Log entries unchanged
3. Only redistribute unchecked items
4. Confirm the new plan with user before saving
5. Update only the Plan section and Profile parameters

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Old log entries are clutter, I'll clean them up" | Log entries are learning history; deleting them destroys the progress record |
| "The user said 'just redo it', I'll start fresh" | Starting fresh loses all progress; always preserve completed records |
| "The plan is obviously wrong, I'll fix it without asking" | "Obviously wrong" to you may be intentional; always confirm changes with the user |

## Red Flags

- You deleted Log entries to "clean up" → they are learning history, never delete
- You adjusted plan without user confirmation → always confirm changes
- User sounds frustrated → acknowledge the difficulty, offer to simplify, don't push through
