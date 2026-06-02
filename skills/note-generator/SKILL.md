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
