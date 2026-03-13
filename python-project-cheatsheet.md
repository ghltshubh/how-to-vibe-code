# Python Project From Scratch — Cheat Sheet

## The 5-Phase Workflow

```
SPEC → PLAN → STRUCTURE → IMPLEMENT → POLISH
```

| Phase | Output | Time |
|-------|--------|------|
| 1. Requirements | Spec (chat or file) | 15-30 min |
| 2. Plan | Task list (chat or file) | 10-15 min |
| 3. Structure | Directory scaffold | 5-10 min |
| 4. Implement | Working code | Varies |
| 5. Polish | Docs + tests | 30 min |

> **Phases 1-2 can be conversational** — no files needed for solo/small projects.

---

## Phase Prompts

### 1. Requirements (conversational)
```
I want to build [PROJECT].
Ask me questions until requirements are clear.
```

### 2. Plan (conversational)
```
Let's plan this out.
Break into numbered steps I can follow.
```

### 3. Structure
```
Create Python project structure (src/ layout).
Include pyproject.toml, tests/, skeleton files.
```

### 4. Implement
```
Implement Step [N]: [DESCRIPTION]
Context: [EXISTING CODE]
Include: type hints, docstrings, error handling, tests.
```

### 5. Review
```
Review this codebase for issues.
Check: consistency, errors, performance, docs, tests.
```

---

## Python Project Structure (src/ layout)

```
project-name/
├── README.md
├── pyproject.toml
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── cli.py
│       ├── core/
│       │   └── main_logic.py
│       ├── io/
│       │   └── readers.py
│       └── utils/
│           └── helpers.py
├── tests/
│   ├── conftest.py
│   └── test_core/
└── docs/
    └── spec.md
```

---

## Implementation Loop

```
┌────────────┐     ┌────────────┐     ┌────────────┐
│ Prompt     │ →   │ Review     │ →   │ Run        │
│ Step N     │     │ code       │     │ tests      │
└────────────┘     └────────────┘     └────────────┘
                                            │
┌────────────┐     ┌────────────┐           │
│ Next       │ ←   │ Commit     │ ←─────────┘
│ step       │     │ changes    │
└────────────┘     └────────────┘
```

---

## Quick Templates

### Spec Sections
```markdown
## Overview
## Requirements (Functional / Non-Functional)
## Data Models (Input / Output)
## Technical Stack
## Testing Strategy
## Edge Cases
```

### Task Format
```
Phase N: [Name] [Small/Medium/Large]
- N.1 [Task description]
- N.2 [Task description]
```

### Implementation Request
```
Implement Step N.M: [description]
Interface: [paste]
Context: [paste related code]
Constraints: [list]
```

---

## Golden Rules

```
1. CLARIFY FIRST  → No code until requirements are clear
2. PLAN THE WORK  → Numbered, trackable tasks
3. SMALL STEPS    → One task per prompt
4. ALWAYS VALIDATE→ "Check for issues" after changes
5. COMMIT OFTEN   → One commit per task
6. PROVIDE CONTEXT→ Show interfaces & constraints
```

---

## Anti-Patterns

| ❌ Don't | ✓ Do |
|----------|------|
| "Build me an ETL pipeline" | Clarify requirements → plan → "Implement Step 1" |
| "Generate entire project" | "Implement Step 3.2: lapse rate correction" |
| Skip validation | "Check for issues" after every change |
| One huge commit | Commit after each task |

---

## Useful Follow-ups

```
"Can you critique the plan for gaps?"
"Check for any issues"
"What about [edge case]?"
"Provide complete corrected file"
"How do I test this?"
"Create documentation for GitHub"
```
