# LLM Prompting Cheat Sheet

## Quick Patterns

| What You Need | Prompt Structure | Example |
|---------------|------------------|---------|
| **Debug** | `[error] + What's wrong?` | Paste traceback |
| **Explain** | `Tell me what this does, I'll ask what next` | Before modifying inherited code |
| **Build** | Start broad → add specifics → validate | `Build DAG` → `Add X` → `Check for issues` |
| **Optimize** | `[constraint] + How to improve?` | "3 pods max, 8 simulations" |
| **Compare** | `Compare [A] with [B]` | Scalable vs non-scalable code |
| **Validate** | `I did [X], check for issues` | After making changes |

---

## The 6 Golden Rules

```
1. CONTEXT FIRST    → Paste code, errors, constraints
2. ONE THING        → One clear ask per message  
3. ITERATE          → Build complexity in follow-ups
4. VALIDATE         → "Check for issues" after changes
5. SHOW STATE       → Terminal output, current config
6. CHALLENGE        → "Why this approach?"
```

---

## Copy-Paste Templates

### Debug
```
I'm getting this error:
[traceback]

Running on: [env]
Last change: [what you did]
```

### New Pipeline
```
Build a pipeline:
- Input: [source, format, size]
- Process: [transforms]
- Output: [format, destination]
- Constraints: [limits]
```

### Review
```
I updated the code to [change].
Check for any issues.
[code]
```

### Optimize
```
Current: [behavior]
Target: [goal]
Constraints: [limits]
What are my options?
```

---

## Anti-Patterns (Avoid)

| ❌ Don't | ✓ Do Instead |
|----------|--------------|
| "Help fix this" | [error] + "What's causing this?" |
| [500 lines] + "improve" | [relevant function] + "optimize this" |
| Skip validation | "Check for issues" after every change |
| Accept first answer | "Why this approach?" |

---

## Conversation Flow

### Iterative Build (ERA5 DAG Example)
```
You: "Build Airflow DAG for ERA5 data"
LLM: [creates DAG]

You: "Add soil_temperature_level_1"
LLM: [adds variable]

You: "Check for issues"
LLM: "Level dimension needs .sel(level=1)"

You: "Provide complete file"
LLM: [full corrected DAG]
```

### Architecture Decision (Downscaling Example)
```
You: "Look at this doc, tell me if you understand"
LLM: [summarizes architecture]

You: "Do I need downscaling for all datasets?"
LLM: [categorizes: ERA5 ✓, CCI ✗]

You: "Rewrite without SAFRAN as special case"
LLM: [generalized architecture]
```

---

## Useful Follow-ups

```
"Now do [next step]"
"What about [edge case]?"
"I did that, check for issues"
"Explain: [specific part]"
"Why this approach?"
"Provide complete file"
```

---

## When to Start New Chat

✓ New project  
✓ Different codebase area  
✓ Previous context irrelevant  

## When to Continue Chat

✓ Building on previous work  
✓ Related debugging  
✓ Same file/module iteration
