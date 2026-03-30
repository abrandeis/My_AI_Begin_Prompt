# 🧠 AI Prompt Versioning & Continuation System

> A structured protocol for maintaining continuity, context, and version control across AI chat sessions.

---

## Overview

This system solves one of the most frustrating problems with AI chat tools: **context death**. Every new chat starts cold. Work, decisions, code patterns, and agreements made in previous sessions vanish. This prompt protocol is designed to eliminate that problem by establishing a repeatable, structured handoff system between sessions.

The core idea is simple: at any point in a working session, you issue a single command and the AI generates everything needed to reconstruct full context in a new chat — no re-explaining, no re-litigating decisions, no starting over.

---

## The Problem It Solves

- AI chats have no memory across sessions
- Complex, multi-session projects lose critical context
- Re-explaining background wastes time and degrades output quality
- No standard format for "picking up where we left off"
- No audit trail of decisions made, patterns established, or next steps agreed upon

---

## How It Works

### Trigger Commands

Issue any of the following phrases to activate the versioning system:

| Command | Action |
|---|---|
| `Update version` | Full version update — summary, changelog, new prompt, helper files |
| `Update Prompt` | Same as above |
| `Update this chat prompt` | Same as above |

### What Gets Generated

When a version update is triggered, the AI produces **four artifacts**:

1. **Summary** — ~450 words covering what was accomplished, with delta notes from the previous version
2. **Change Log** — Downloadable in both `.txt` and `.html` format, with timestamps and version numbers
3. **Continuation Prompt** — A `.txt` file containing the full prompt needed to re-open a new chat with complete context
4. **Helper Files** — Supporting context in `JSON`, `HTML`, or `TXT` format (whichever is most efficient) for the AI to parse rapidly on re-entry

> **Note:** The prompt regeneration is **never** included in a normal response. It is only produced on an explicit `Update Version` request.

---

## Output File Structure

```
/session-exports/
├── v1.0_summary.txt          # Human-readable session summary
├── v1.0_changelog.html       # Formatted changelog with versioning
├── v1.0_changelog.txt        # Plain text changelog
├── v1.0_continuation.txt     # The re-entry prompt for next session
└── v1.0_context.json         # Machine-readable helper context
```

---

## Version Log Format

Each version entry in the changelog includes:

- **Version number** (e.g., `v1.0`, `v1.1`, `v2.0`)
- **Date/time** of the update (when available)
- **Summary** of the session at the top
- **Key decisions made**
- **Code patterns established**
- **Next steps identified**
- **Suggestions agreed upon**
- **Any additions or changes** from the prior version

---

## Versioning Convention

| Version Type | When to Use |
|---|---|
| `v1.0 → v1.1` | Minor additions, clarifications, small feature iterations |
| `v1.x → v2.0` | Major scope changes, architectural pivots, new modules added |

---

## Re-Entry Protocol

To resume a session in a new chat:

1. Open the saved `v[X.X]_continuation.txt` file
2. Paste its full contents as your **first message** in the new chat
3. Attach any relevant `v[X.X]_context.json` helper files if referenced
4. The AI will confirm it has absorbed the context and is ready to continue

---

## Design Goals

- **Speed** — JSON helper files allow faster AI parsing vs. long prose prompts
- **Fidelity** — Detailed changelogs prevent decision drift across sessions
- **Portability** — Plain text formats work across any AI tool or interface
- **Auditability** — Every version is a recoverable snapshot of the project state

---

## Current Version

`v1.0` — Initial system definition. Project scope, structure, and trigger commands established. Underlying project (referenced in the prompt body) is pending scoping questions before first working session begins.

---

## Notes

- Ask clarifying questions **before** completing a version update if context is ambiguous
- Helper files can be in any format — use whatever is most machine-readable
- The system is tool-agnostic: works with Claude, GPT, Gemini, or any chat-based AI
- This `README.md` itself should be versioned alongside the prompt files

---

*Built for multi-session AI workflows where context continuity is non-negotiable.*
