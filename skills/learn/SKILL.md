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
