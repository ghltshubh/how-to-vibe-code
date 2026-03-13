# LLM Prompting Guide for Coding Exercises

A practical guide for getting the best results when working with AI on data engineering and geospatial coding tasks. Examples drawn from real downscaling pipeline, ERA5 DAG, and wildfire simulation projects.

---

## Quick Reference Cheat Sheet

### Prompt Pattern Matrix

| Goal | Prompt Pattern | Example |
|------|----------------|---------|
| **Debug an error** | Paste error + "what's wrong?" | `ValueError: cannot specify both keyword arguments` |
| **Explain code** | "Tell me what this code is doing" + paste code | Wildfire simulation pipeline walkthrough |
| **Iterative build** | Start broad → narrow with follow-ups | "Build ERA5 DAG" → "Add soil temp" → "Fix level dim" |
| **Compare approaches** | "Compare X with Y" or show two versions | Scalable vs non-scalable data processing |
| **Optimize** | State constraints + ask for options | "3 pods max, 8 simulations. How to optimize?" |
| **Documentation** | "Summarize what we did + create docs" | Daily updates + GitHub README |
| **Review/validate** | "I did X, check for any issues" | Verify code after user makes changes |
| **Architecture** | "Do I need X for Y?" + provide context | "Do I need downscaling for each dataset?" |

### Golden Rules

```
1. CONTEXT FIRST    → Paste relevant code, errors, or constraints
2. ONE THING        → Each message should have one clear ask
3. ITERATE          → Build up complexity through follow-up prompts
4. VALIDATE         → "Check for issues" after you make changes
5. SHOW STATE       → When debugging, show current state (terminal output)
6. CHALLENGE        → "Why are we using X?" - question assumptions
```

---

## Part 1: Effective Prompt Patterns

### Pattern 1: Debug Mode

**When to use:** You have an error and need help understanding/fixing it.

**Structure:**
```
[Paste error traceback]

What's wrong here?
```

**Real Example (Downscaling Pipeline - Zarr merge):**
```
Error during concatenation: cannot specify both keyword and positional arguments to .isel
Traceback (most recent call last):
  File "merge_batches.py", line 435, in concat_zarr_files_new_dimension
    sample_data = verification.isel(
ValueError: cannot specify both keyword and positional arguments to .isel
```

**Why it works:** AI immediately sees the exact error and can trace the root cause.

---

### Pattern 2: Explain Mode

**When to use:** You inherited code or need to understand complex logic.

**Structure:**
```
Can you tell me what this code is doing and I will ask you what to do next
[Paste code]
```

**Real Example (Wildfire Pipeline):**
```
Can you tell me what this code is doing and I will ask you what to do next

class WildfireSimulationPipeline:
    def run_data_acquisition(self):
        # Downloads Sentinel-2, computes indices, loads terrain...
```

**Why it works:** Gets comprehensive understanding first, then you can direct the next steps.

---

### Pattern 3: Iterative Build

**When to use:** Building something new (DAG, pipeline, module).

**Workflow:**
```
Step 1: High-level ask     → "Build an Airflow DAG to download ERA5 data"
Step 2: Add specifics      → "Add soil_temperature_level_1 to the variables"
Step 3: Handle edge cases  → "Soil temp has a 'level' dimension, select level 1"
Step 4: Validate           → "I updated the code, check for any issues"
```

**Real Example (ERA5 DAG Build):**
```
Session flow:
├── "Build an Airflow DAG to download ERA5 ARCO data for fire spread modeling"
│   └── AI creates initial DAG with 6 default variables
├── "Why are we using such a huge decorator with all those params?"
│   └── AI explains Airflow params pattern vs hardcoding
├── "I added soil_temperature_level_1, check for any issues"
│   └── AI catches: "Soil temp has 'level' dimension - needs .sel(level=1)"
└── "Provide the complete corrected file"
    └── AI outputs full working DAG
```

---

### Pattern 4: Constrained Optimization

**When to use:** Need to improve performance within hard limits.

**Structure:**
```
[Describe the constraint]
[Current behavior]
Is there any way to [goal] without [constraint violation]?
```

**Real Example (Wildfire Simulation):**
```
This code takes longer time on a Kubernetes cluster when we run 8 simulations. 
We can only get 3 pods max on Kubernetes. 
Is there any way to optimize this code so that it runs faster 
without acquiring additional resources?
```

**Why it works:** Clear constraints prevent AI from suggesting impossible solutions.

---

### Pattern 5: Architecture Validation

**When to use:** Making design decisions, understanding scope.

**Structure:**
```
Look at this document and tell me if you understand this
[Paste document/diagram]
```

Then follow up with:
```
Do you think I will need [X] for all the other [Y] too?
```

**Real Example (Downscaling Pipeline):**
```
Session flow:
├── "Look at this document about climate data downscaling"
│   └── AI summarizes: Two-stage architecture, grid alignment, etc.
├── "Do you think I will need downscaling for all the other datasets too?"
│   └── AI categorizes: ERA5-Land ✓, DRIAS ✓, CCI Biomass ✗ (already 100m)
└── "Can you rewrite this but not considering SAFRAN as a special case"
    └── AI revises recommendation with generalized pipeline
```

---

### Pattern 6: Configuration Discovery

**When to use:** Trying to find optimal parameters.

**Structure:**
```
I want to [goal]. What config of input parameters should I use?
Ask for any information that you need.
Currently I am running [current command]
but I think [observation]
```

**Real Example (Zarr Processing):**
```
I want to write the output to the disk ASAP and not hold any data in memory. 
What config of input parameters should I use. Ask for any information that you need.
Currently I am running:
`python stream_scaled_preprocessing_v3.py --batch-size 144 --var-batch-size 2`
but I think its trying to read all variables
```

**Why it works:** Invites AI to ask clarifying questions and understand context.

---

### Pattern 7: Quick Fix

**When to use:** Simple syntax or command issues.

**Structure:**
```
[Paste error or command]
```

**Real Examples:**
```
# Shell quoting issue
Human: pip install gcsfs>=2023.1.0
       zsh: 2023.1.0 not found
LLM: Quote it: pip install "gcsfs>=2023.1.0"

# Circular import
Human: [Paste ImportError traceback]
LLM: Remove line 11 - file is importing from itself
```

---

### Pattern 8: Documentation Generation

**When to use:** Need daily updates, READMEs, or summaries.

**Structure:**
```
Give me short to-do for daily updates to the team example:
Today: 
1. ...
Tomorrow:
1. ...

And also summarize what we did and create a short documentation for GitHub
```

---

## Part 2: Anti-Patterns to Avoid

### ❌ Vague Requests
```
Bad:  "Help me fix this"
Good: [Paste error] + "What's causing this?"
```

### ❌ Information Overload
```
Bad:  [Paste entire 500-line file] + "Make it better"
Good: [Paste relevant function] + "This function is slow, how can I optimize?"
```

### ❌ Skipping Validation
```
Bad:  Make changes → Ship to production
Good: Make changes → "I did X, check for issues" → Fix → Ship
```

### ❌ Not Challenging Assumptions
```
Bad:  Accept first solution
Good: "Why are we using such a huge decorator?" → Understand rationale
```

### ❌ Losing Context
```
Bad:  [3 messages of code] then "does this help" with no context
Good: "Does this diagram help clarify the input format question?"
```

---

## Part 3: Downscaling Pipeline Session Walkthrough

Here's how a real multi-turn session builds a downscaling pipeline:

```
┌─────────────────────────────────────────────────────────────┐
│ PHASE 1: CONTEXT LOADING                                    │
├─────────────────────────────────────────────────────────────┤
│ Human: "This is an export for JIRA tickets... Got feedback" │
│ Human: [Uploads climate data pipeline document]             │
│ Human: "Look at this document and tell me if you understand"│
│                                                             │
│ LLM: Summarizes key concepts:                            │
│   - Two-stage architecture (NetCDF → iLand)                │
│   - Grid alignment requirements                             │
│   - Variables needed (temp, precip, radiation, VPD, wind)   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ PHASE 2: SCOPE CLARIFICATION                                │
├─────────────────────────────────────────────────────────────┤
│ Human: "Do you think I will need downscaling for all the   │
│         other datasets too?"                                │
│                                                             │
│ LLM: Categorizes datasets:                               │
│   ✓ ERA5-Land (~11km) - YES                                │
│   ✓ DRIAS (8km) - YES                                      │
│   ✓ DestinE (~4-5km) - YES                                 │
│   ✗ CCI Biomass (100m) - Already at target resolution      │
│   ✗ CHELSA-W5E5 - Missing humidity/wind                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ PHASE 3: DESIGN ITERATION                                   │
├─────────────────────────────────────────────────────────────┤
│ Human: "Can you rewrite this but not considering SAFRAN as │
│         a special case"                                     │
│                                                             │
│ LLM: Revises to generalized architecture:                │
│   Climate Pipelines → Generalized Downscaling (4-11km→100m)│
│                    → Stage 1: Unified NetCDF                │
│                    → Stage 2: iLand format converters       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ PHASE 4: IMPLEMENTATION DETAIL                              │
├─────────────────────────────────────────────────────────────┤
│ Human: "Where did you get the input format netcdf from?"   │
│                                                             │
│ LLM: Explains assumption, asks for clarification         │
│                                                             │
│ Human: "Look at the data [URL]"                            │
│                                                             │
│ LLM: Fetches URL, discovers actual format (CSV/GRIB)     │
│         Updates config accordingly                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 4: Prompt Templates

### Template: New Pipeline Request
```
I need to build a pipeline that:
- Input: [data source, format, size]
- Process: [transformations needed]
- Output: [target format, destination]
- Constraints: [memory, compute, scheduling]

Can you help me design the architecture?
```

### Template: Debug Session
```
I'm getting this error:
[Paste full traceback]

Context:
- Running on: [environment]
- Input data: [description]
- Last thing I changed: [what you modified]
```

### Template: Code Review Request
```
I updated the code to [describe change].
Check for any issues.

[Paste relevant code section]
```

### Template: Performance Optimization
```
Current behavior: [what's happening]
Target: [what you want]
Constraints: [resources available]
Bottleneck: [if known]

What are my options?
```

### Template: Architecture Decision
```
Context: [project background]
Question: Should I use [Option A] or [Option B]?
Considerations:
- [Factor 1]
- [Factor 2]

What would you recommend and why?
```

---

## Part 5: Conversation Management

### When to Start a New Chat
- New project or pipeline
- Different area of the codebase
- Context from previous chat no longer relevant

### When to Continue Same Chat
- Building on previous work
- Debugging related issues
- Iterating on same file/module

### Effective Follow-ups
```
✓ Good:
├── "Now do [next step]"
├── "What about [edge case]?"
├── "I did that, check for issues"
└── "Explain: [specific part you don't understand]"

✗ Less effective:
├── "Thanks!" (no follow-up needed)
├── "Can you also do [unrelated thing]?" (start new chat)
└── [Long silence then context-dump] (AI loses context)
```

---

## Appendix: Real Prompts from Projects

### ERA5 Pipeline Session
```
"Build an Airflow DAG to download ERA5 ARCO data for fire spread modeling"
"Why are we using such a huge decorator?"
"I added soil_temperature_level_1, check for any issues"
"Provide the complete corrected file"
"How do I test it?"
```

### Wildfire Optimization Session
```
"Can you tell me what this code is doing and I will ask what to do next"
"This takes longer on K8s with 8 simulations. 3 pods max. How to optimize?"
```

### Zarr Processing Session
```
"I want to write output to disk ASAP and not hold data in memory"
"Will using 'a' instead of 'r+' help?"
"Remove emojis" (formatting preference)
```

### COSIA Pipeline Session
```
"Now create a test DAG that converts the data into PostGIS and rasters"
"The data in the blobs isn't changing anymore, let's assume"
```

### Downscaling Scope Session
```
"Look at this document and tell me if you understand"
"Do you think I will need downscaling for all the other datasets too?"
"Can you rewrite this but not considering SAFRAN as a special case"
"Where did you get the input format netcdf from?"
"Look at the data [URL]"
```
