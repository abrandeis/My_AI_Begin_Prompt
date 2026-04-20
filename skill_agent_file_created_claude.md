-----

## name: session-snapshot
description: >
Serializes the current conversation into portable, resumable files so the user
can continue exactly where they left off in a new chat. Trigger this skill
whenever the user types one of these exact commands: /snapshot, /lock, /update-prompt,
[SNAPSHOT], [LOCK VERSION], or says “update version”, “update prompt”, “lock version”,
“update this chat prompt”. Also trigger when loading a prior session with /resume,
[RESUME], or “resume from snapshot”. This skill must always be used for these commands —
never handle them as casual conversation.

# Session Snapshot Skill

Captures the full state of a conversation into 4 output files the user can download,
store, and use to resume the session precisely in a new chat. Also handles loading
a prior snapshot to restore context.

-----

## Trigger Commands

### Save / Update

|Command                                                                                       |Action                                                    |
|----------------------------------------------------------------------------------------------|----------------------------------------------------------|
|`/snapshot`                                                                                   |Create or update all 4 output files                       |
|`/lock`                                                                                       |Same as /snapshot — use when “locking” a milestone version|
|`/update-prompt`                                                                              |Same as /snapshot                                         |
|`[SNAPSHOT]`, `[LOCK VERSION]`                                                                |Bracket-style equivalents                                 |
|Natural language: “update version”, “update prompt”, “lock version”, “update this chat prompt”|Same                                                      |

### Load / Resume

|Command                                                     |Action                                                                  |
|------------------------------------------------------------|------------------------------------------------------------------------|
|`/resume`                                                   |User will paste or attach prior snapshot JSON — load and restore context|
|`[RESUME]`                                                  |Same                                                                    |
|Natural language: “resume from snapshot”, “load my snapshot”|Same                                                                    |

-----

## On Trigger: Confirm Before Acting

When a save command is detected, **do not immediately generate output**.
First respond with a brief confirmation block:

```
📸 Snapshot requested.
  Current version: X.Y
  New version will be: X.Z
  Topics to capture: [1–3 sentence summary of what will be snapshotted]

Proceed? (yes / adjust scope first)
```

Only generate the 4 output files after the user confirms.

-----

## Version Numbering Rules

- **Minor increment** (x.0 → x.1): Normal snapshot update, continuing same thread
- **Major increment** (1.x → 2.0): User explicitly says “new phase”, “starting fresh section”, or topic shifts substantially
- Version is tracked inside the JSON state file (`meta.version`)
- Always display current → new version in the confirmation step
- First snapshot in a conversation is always `1.0`

-----

## Output: 4 Required Files

Every snapshot produces exactly these 4 files. Name them with the version number.

### File 1 — Continuation Prompt (`prompt_vX.Y.txt`)

A standalone text prompt the user pastes into a new chat to resume.
It must be **self-contained** — Claude reading it cold should be able to reconstruct
full context without any other file.

Structure:

```
## RESUME PROMPT — Session: [topic] | Version X.Y | [Date]

### Context
[2–3 sentence description of who the user is, what they're working on, and why]

### What We Established
[Bulleted list: decisions made, patterns set, code/approaches agreed on]

### Where We Left Off
[Precise description of the last thing completed and the immediate next step]

### Open Items
[Numbered list of unresolved questions or pending tasks]

### Operating Rules for This Session
[Any behavioral instructions the user set: tone, step-by-step validation, etc.]

### How to Use This Prompt
Paste this prompt at the start of a new chat. Then attach snapshot_vX.Y.json
for full structured context. Say /resume to confirm context loaded.
```

-----

### File 2 — State File (`snapshot_vX.Y.json`)

Machine-readable structured context. Claude reads this on /resume to restore state
faster than re-reading prose. **Schema is fixed** — always use exactly these fields:

```json
{
  "meta": {
    "version": "X.Y",
    "created": "ISO8601 timestamp",
    "updated": "ISO8601 timestamp",
    "session_topic": "Short title",
    "total_snapshots": 1
  },
  "context": {
    "user_goal": "Primary objective in 1–2 sentences",
    "domain": "e.g. software development / SAT prep / financial research",
    "constraints": ["list", "of", "constraints or rules established"],
    "operating_mode": "e.g. step-by-step validation before advancing"
  },
  "state": {
    "completed": [
      { "id": 1, "description": "What was finished", "outcome": "result or decision" }
    ],
    "in_progress": {
      "description": "What is currently being worked on",
      "last_action": "The last thing done",
      "next_action": "The immediate next step"
    },
    "blocked": [
      { "id": 1, "description": "What is blocked", "reason": "Why" }
    ]
  },
  "decisions": [
    { "id": 1, "decision": "What was decided", "rationale": "Why", "version_set": "X.Y" }
  ],
  "open_items": [
    { "id": 1, "item": "Description", "priority": "high|medium|low" }
  ],
  "version_history": [
    { "version": "1.0", "date": "ISO8601", "summary": "What changed in this version" }
  ],
  "files": {
    "prompt": "prompt_vX.Y.txt",
    "snapshot": "snapshot_vX.Y.json",
    "log_txt": "log_vX.Y.txt",
    "log_html": "log_vX.Y.html"
  }
}
```

**Do not add or remove top-level keys.** Extend within existing arrays only.
This keeps the schema stable across versions so future Claude instances can parse it reliably.

-----

### File 3 — Change Log Text (`log_vX.Y.txt`)

Plain text log. Cumulative — each new version appends, older versions stay.

Structure:

```
SESSION LOG — [Topic] 
================================================

[VERSION X.Y] — [Date/Time if available]
------------------------------------------------
SUMMARY
[3–5 sentence narrative of what this session covered. Not a list — prose.]

KEY DECISIONS
- [Decision and rationale]

CODE / PATTERNS ESTABLISHED  
- [Any code patterns, conventions, or technical choices locked in]

NEXT STEPS
- [Ordered list of immediate next actions]

SUGGESTIONS AGREED ON
- [Anything proposed by Claude and accepted by user]

OPEN QUESTIONS
- [Unresolved items carried forward]

------------------------------------------------
[VERSION X.Z — previous version block below]
...
```

-----

### File 4 — Change Log HTML (`log_vX.Y.html`)

Same content as File 3 but rendered as clean, readable HTML.
Requirements:

- Self-contained single file (inline CSS, no external deps)
- Version sections as collapsible `<details>` blocks, newest expanded by default
- Color-coded sections: decisions (blue), next steps (green), open questions (amber)
- Header shows: topic, current version, last updated date
- Dark background preferred (`#1a1a2e` or similar) — readable on any device

-----

## On /resume: Loading a Prior Snapshot

When the user triggers `/resume`:

1. Ask: “Please paste your snapshot JSON or attach `snapshot_vX.Y.json`”
1. Parse the JSON using the schema above
1. Respond with a structured confirmation:

```
✅ Snapshot loaded — Version X.Y
Topic: [session_topic]
Last action: [state.in_progress.last_action]
Next action: [state.in_progress.next_action]
Open items: [count] items pending

Ready to continue. Starting with: [next_action]
```

1. Proceed immediately into the session — do not re-explain what’s in the snapshot unless asked
1. Operating rules from `context.operating_mode` and `context.constraints` are now active

-----

## Edge Cases

|Situation                                     |Behavior                                                                   |
|----------------------------------------------|---------------------------------------------------------------------------|
|First snapshot in a conversation              |Version = 1.0, `version_history` has one entry                             |
|User snapshots twice without new work         |Increment version anyway, note “minor update / no material changes” in log |
|No date/time available                        |Use “Date unavailable” — do not fabricate                                  |
|JSON file not provided on /resume             |Work from pasted prompt text only; note state file absent                  |
|User says “adjust scope first” at confirmation|Ask what to include/exclude, then regenerate confirmation                  |
|Topic is sensitive or personal                |Summarize at the level of detail the user established — do not editorialize|

-----

## Quality Checks Before Delivering Files

Before outputting the 4 files, verify:

- [ ] Prompt file is self-contained — a cold Claude could resume from it alone
- [ ] JSON validates against the schema (all required keys present)
- [ ] Log is cumulative (prior versions not overwritten)
- [ ] HTML renders without external dependencies
- [ ] Version number is consistent across all 4 files
- [ ] Confirmation was shown and user approved before this point