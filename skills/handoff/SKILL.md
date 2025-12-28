---
name: handoff
description: Manages context transfer between AI coding sessions. Activates when HANDOFF.md exists, when user mentions handoff/resume, or when ending significant work.
---

# Handoff Detection

## On Session Start

Check if `HANDOFF.md` exists in the working directory. If found:

1. Read it silently
2. Tell the user: "Found a handoff from a previous session: [title]. [1-sentence goal]. Resume from here?"
3. If they agree, follow `/handoff:resume` flow

## Trigger Words

Activate when user says: "handoff", "hand off", "pass this to", "continue later", "pick up where", "transfer context", "save state", "resume", "take over"

## Creating vs Resuming

- User wants to **create**: They're wrapping up or switching agents → use `/handoff:create`
- User wants to **resume**: They're starting fresh with existing handoff → use `/handoff:resume`

## Proactive Suggestions

Consider suggesting a handoff when:
- User says "I need to go" or "let's stop here"
- A significant milestone is reached
- You've been working for a long time with lots of context

Say: "Want me to create a handoff so you (or another agent) can continue later?"

## Commands

| Command | Use When |
|---------|----------|
| `/handoff:create` | Full handoff with all context |
| `/handoff:quick` | Minimal handoff, just essentials |
| `/handoff:resume` | Continue from existing handoff |
