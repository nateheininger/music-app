# Project Docs — How to Use This System

> Read this once. The system only works if you actually use it.

## What these docs are for

This is the project's **long-term memory**. AI conversations are short-lived and disposable; these docs are permanent. When you start a new conversation with Claude (or any other collaborator), you paste in the relevant docs and they get up to speed in minutes instead of hours.

## The five docs

| Doc | Purpose | Update frequency |
|---|---|---|
| `PROJECT.md` | The north star. What we're building, who for, what's in/out of scope. | Rarely |
| `ARCHITECTURE.md` | The technical map. Stack, components, data model. | When architecture changes |
| `DECISIONS.md` | Append-only log of *why* we made each major choice. | Every meaningful decision |
| `BACKLOG.md` | What's next, what's deferred. The dam against scope creep. | Frequently |
| `OPEN_QUESTIONS.md` | Things we haven't figured out yet. | When questions arise/resolve |

## The conversation pattern that works

**Start each AI conversation with context:**
> "Working on the TAB playlist sync project. Here's the current state: [paste PROJECT.md and the relevant section of ARCHITECTURE.md]. Today I want to work on [specific thing]."

**Keep each conversation focused on one chunk of work.** Examples of right-sized scopes:
- Designing the database schema
- Implementing one specific OAuth flow
- Debugging a specific bug
- Code-reviewing a PR
- Resolving one open question

**End each conversation with an artifact:**
> "Summarize the key decisions from this conversation and give me the diff to add to DECISIONS.md and ARCHITECTURE.md."

Then **actually update the docs.** This is the part that matters most. The conversation is ephemeral; the docs are your memory.

## When to start a fresh conversation

- Topic shift (e.g., done debugging, now want to design something new)
- Conversation is getting slow or AI is making mistakes it wouldn't have earlier
- Returning after a few days
- About to ask something fundamentally different

Rule of thumb: more than ~30-40 substantive back-and-forths = time to wrap up and start fresh.

## The trap to avoid

Don't treat the AI conversation as the project memory. Don't keep one giant chat going for weeks scrolling up to find what you decided. That breaks down because chats get unwieldy, decisions get buried, and you can't reproduce the state when something goes wrong.

**The repo's docs are the project memory. The AI is a thinking partner you bring in for specific tasks.**

## Maintenance discipline

- After every meaningful AI conversation, ask for doc updates and apply them.
- Once a week or so, skim BACKLOG.md and OPEN_QUESTIONS.md to keep them current.
- Once a month, re-read PROJECT.md and ask yourself: "is this still what I'm building?" If it's drifted, update it.

## File locations

These docs live at the root of the repo (or in a `/docs` folder, your call). Commit them to git like any other source. Their history is part of the project's history.
