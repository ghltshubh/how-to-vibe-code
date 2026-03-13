# Building Python Projects From Scratch with LLMs

A structured workflow for using LLMs to plan, design, and implement Python projects. Based on industry best practices and real-world patterns.

---

## The Core Philosophy

> "Don't just throw wishes at the LLM — begin by defining the problem and planning a solution."
> — Addy Osmani

The key insight: **Planning first forces you and the AI onto the same page and prevents wasted cycles.** Experienced developers treat a robust spec/plan as the cornerstone of the workflow.

---

## Quick Reference: The 5-Phase Workflow

| Phase | Output | Key Prompt |
|-------|--------|------------|
| 1. Requirements | Spec (in chat or file) | "Ask me questions until we've fleshed out requirements" |
| 2. Project Plan | Task list (in chat or file) | "Break this into logical, numbered tasks" |
| 3. Structure | Directory scaffold | "Create project structure with src/ layout" |
| 4. Implementation | Working code | "Let's implement Step N from the plan" |
| 5. Polish | Docs + tests | "Review for issues, add documentation" |

> **Note:** Phases 1-2 don't require actual files — the spec and plan can live entirely in your conversation. Only create `spec.md` / `plan.md` files for team projects or complex systems you'll reference later.

---

## Phase 1: Requirements & Specification

### The Goal
Clarify requirements through Q&A before writing any code. This can be a conversation or a formal `spec.md` file.

### Two Approaches

**Quick (in-conversation):**
```
Human: I want to build X. Ask me questions.
LLM:   [asks 5-7 clarifying questions]
Human: [answers]
LLM:   Great, here's what we're building: [summary]
       Ready to start planning?
```

**Formal (create spec.md):**
```
Human: I want to build [PROJECT].
       Ask me questions until we've fleshed out requirements.
       Then compile into a spec document.
```

### Prompt Template
```
I want to build [PROJECT DESCRIPTION].

Please ask me questions iteratively until we've fleshed out:
- Core requirements and features
- Edge cases and constraints  
- Data models and interfaces
- Technical stack preferences
- Testing strategy

After our discussion, compile everything into a comprehensive spec.md
```

### Real Example (Downscaling Pipeline)
```
Human: I want to build a climate data downscaling pipeline that takes 
       8km gridded data and produces 100m resolution outputs for the 
       iLand forest model.

LLM: Great! Let me ask some clarifying questions:

1. What input formats are you working with? (NetCDF, GRIB, Zarr?)
2. What variables need to be downscaled? (temp, precip, wind, etc.)
3. What's the spatial extent? (France? Europe?)
4. What's the expected data volume per run?
5. Should this be a one-time script or a scheduled pipeline (Airflow)?
6. What cloud infrastructure are you using? (Azure, GCP, local?)
7. Are there existing tools/libraries you need to integrate with?

[Iterative Q&A continues...]
```

### What Makes a Good Spec
```
# Project Specification: Climate Downscaling Pipeline

## Overview
One-paragraph description of what we're building and why.

## Requirements
### Functional
- FR-1: Accept NetCDF/GRIB inputs from SAFRAN, ERA5-Land, DRIAS
- FR-2: Apply bilinear interpolation + lapse rate correction
- FR-3: Output unified NetCDF at 100m aligned to forest grid

### Non-Functional
- NFR-1: Process full France dataset in < 2 hours
- NFR-2: Memory usage < 32GB for standard runs
- NFR-3: Graceful handling of corrupt input files

## Data Models
### Input Schema
- Variables: temperature, precipitation, humidity, wind_u, wind_v
- Dimensions: time (hourly), lat, lon
- Coordinate system: WGS84

### Output Schema
- Same variables, 100m LAMB93 grid
- Aligned to forest mask resource units

## Technical Stack
- Python 3.11+
- xarray + dask for data handling
- rasterio for raster operations
- Airflow for scheduling

## Testing Strategy
- Unit tests for each module
- Integration test with sample 10x10km region
- Validation against météo station observations

## Edge Cases
- France crosses 0° meridian — handle longitude wrapping
- Missing data — fill with temporal interpolation
- Corrupt files — log and skip, don't fail entire run
```

---

## Phase 2: Project Plan

### The Goal
Break the work into numbered, trackable tasks. This can stay in the conversation or be saved as `plan.md`.

### Quick Approach
```
Human: OK, let's plan this out. Break into numbered steps.
LLM:   Here's the plan:
       1. Setup project structure
       2. Implement data loader
       3. Add core processing logic
       ...
       Ready to start with Step 1?
Human: Yes, let's go.
```

### Formal Prompt Template
```
Based on this spec, generate a project plan:

[PASTE spec.md]

Break the implementation into:
1. Logical phases or milestones
2. Specific tasks within each phase
3. Dependencies between tasks
4. Estimated complexity (small/medium/large)

Format as a numbered task list I can execute step by step.
```

### Example Output
```markdown
# Project Plan: Climate Downscaling Pipeline

## Phase 1: Project Setup [Small]
- 1.1 Create project structure with src/ layout
- 1.2 Set up pyproject.toml with dependencies
- 1.3 Configure logging and basic CLI
- 1.4 Initialize git repository

## Phase 2: Data Loaders [Medium]
- 2.1 Implement NetCDF reader for ERA5-Land format
- 2.2 Implement GRIB reader for SAFRAN format
- 2.3 Create unified DataLoader interface
- 2.4 Write unit tests for loaders

## Phase 3: Core Downscaling [Large]
- 3.1 Implement bilinear interpolation module
- 3.2 Add lapse rate correction for temperature
- 3.3 Add terrain correction for radiation
- 3.4 Create downscaling orchestrator
- 3.5 Handle longitude wrapping for France
- 3.6 Write integration tests with sample data

## Phase 4: Output & Integration [Medium]
- 4.1 Implement NetCDF writer with CF conventions
- 4.2 Add Airflow DAG wrapper
- 4.3 Add Azure Blob upload functionality
- 4.4 End-to-end testing

## Phase 5: Documentation & Polish [Small]
- 5.1 Write README with usage examples
- 5.2 Add docstrings and type hints throughout
- 5.3 Create example notebooks
- 5.4 Final code review
```

### Refining the Plan
```
Human: Can you critique the plan for any gaps?

LLM: A few things I'd add:
1. Phase 2 should include validation of coordinate systems
2. Phase 3 needs explicit task for VPD calculation from humidity
3. Missing: error handling strategy for corrupt input files
4. Consider adding data profiling step before implementation
```

---

## Phase 3: Project Structure

### The Goal
Create a clean, scalable directory layout before writing any code.

### Recommended Python Structure (src/ layout)
```
project-name/
├── README.md
├── LICENSE
├── pyproject.toml              # Modern Python packaging
├── .gitignore
├── .pre-commit-config.yaml     # Code quality hooks
│
├── src/
│   └── project_name/           # Importable package
│       ├── __init__.py
│       ├── cli.py              # Command-line interface
│       ├── config.py           # Configuration handling
│       │
│       ├── core/               # Core business logic
│       │   ├── __init__.py
│       │   ├── downscaler.py
│       │   └── interpolation.py
│       │
│       ├── io/                 # Input/Output modules
│       │   ├── __init__.py
│       │   ├── readers.py
│       │   └── writers.py
│       │
│       └── utils/              # Shared utilities
│           ├── __init__.py
│           └── logging.py
│
├── tests/                      # Test suite
│   ├── __init__.py
│   ├── conftest.py             # Pytest fixtures
│   ├── test_core/
│   │   └── test_downscaler.py
│   └── test_io/
│       └── test_readers.py
│
├── docs/                       # Documentation
│   ├── spec.md
│   └── plan.md
│
├── scripts/                    # One-off scripts
│   └── profile_data.py
│
└── notebooks/                  # Jupyter notebooks
    └── exploration.ipynb
```

### Why src/ Layout?
- Prevents import confusion during development
- Forces you to install the package (`pip install -e .`)
- Tests run against installed version, not local files
- Used by pandas, FastAPI, pytest, and other major projects

### Prompt to Generate Structure
```
Based on this project plan, create the directory structure.

[PASTE plan.md]

Requirements:
- Use src/ layout for proper packaging
- Include tests/ directory mirroring src/ structure
- Add pyproject.toml with ruff for linting, pytest for testing
- Create skeleton files with type hints and docstrings
- Define interfaces/protocols before implementations

Output the full file tree and contents of key files.
```

---

## Phase 4: Iterative Implementation

### The Core Principle

LLMs do best when given focused prompts: implement one function, fix one bug, add one feature at a time.

### The Loop
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │  Prompt  │ →  │  AI  │ →  │  Review  │         │
│   │  Step N  │    │  codes   │    │  code    │         │
│   └──────────┘    └──────────┘    └──────────┘         │
│                                        │               │
│                                        ▼               │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │  Next    │ ←  │  Commit  │ ←  │  Run     │         │
│   │  step    │    │  changes │    │  tests   │         │
│   └──────────┘    └──────────┘    └──────────┘         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Prompt Template for Implementation
```
Let's implement Step [N] from the plan: [TASK DESCRIPTION]

Context:
- Here's the interface it should implement: [PASTE INTERFACE]
- It integrates with: [PASTE RELATED CODE]
- Constraints: [REQUIREMENTS]

Please implement with:
- Type hints
- Docstrings
- Error handling
- Unit tests
```

### Real Session Example
```
Human: Let's implement Step 2.1: NetCDF reader for ERA5-Land format

Here's the interface:
```python
class DataLoader(Protocol):
    def load(self, path: Path) -> xr.Dataset: ...
    def validate(self, ds: xr.Dataset) -> bool: ...
```

It should handle:
- Lazy loading with dask chunks
- Variable selection
- Time slicing

LLM: Here's the implementation:

```python
# src/climate_downscaler/io/readers.py
from pathlib import Path
import xarray as xr

class ERA5LandReader:
    """Reader for ERA5-Land NetCDF/Zarr files."""
    
    REQUIRED_VARS = ["2m_temperature", "total_precipitation"]
    
    def __init__(self, chunk_sizes: dict | None = None):
        self.chunk_sizes = chunk_sizes or {"time": 24}
    
    def load(self, path: Path) -> xr.Dataset:
        """Load ERA5-Land data with lazy dask arrays."""
        ds = xr.open_zarr(path, chunks=self.chunk_sizes)
        return self._standardize_coords(ds)
    
    def validate(self, ds: xr.Dataset) -> bool:
        """Verify required variables exist."""
        return all(v in ds for v in self.REQUIRED_VARS)
```
### Validation After Each Step
```
Human: I implemented the reader. Check for any issues.

LLM: A few things I noticed:
1. Missing handling for when path doesn't exist
2. Should validate coordinate names (latitude vs lat)
3. No logging — hard to debug in production
4. Consider adding a context manager for resource cleanup
```

### Commit Often
After each successful implementation:
```bash
git add src/climate_downscaler/io/readers.py tests/test_io/test_readers.py
git commit -m "feat(io): add ERA5-Land NetCDF reader with lazy loading"
```

---

## Phase 5: Review & Polish

### Code Review Prompt
```
Review the entire codebase for:
1. Consistency issues
2. Missing error handling
3. Performance concerns
4. Documentation gaps
5. Test coverage holes

Here's the current state:
[PASTE DIRECTORY LISTING OR KEY FILES]
```

### Documentation Generation
```
Generate README.md for this project with:
- Project description
- Installation instructions
- Quick start example
- API reference for main functions
- Contributing guidelines

Based on: [PASTE spec.md and key module code]
```

### Final Checklist Prompt
```
Before shipping, verify:
- [ ] All tests pass
- [ ] Type hints on public functions
- [ ] Docstrings on public functions
- [ ] README with installation and usage
- [ ] No hardcoded paths or credentials
- [ ] Error messages are helpful
- [ ] Logging is configured

Review this codebase and flag any violations.
```

---

## Complete Session Example

### Minimal Session (Solo/Small Project)

For quick projects, planning can be just 2-3 exchanges:

```
Human: I want to build a script that downloads ERA5 data. Ask me questions.
LLM:   1. What variables? 2. Time range? 3. Output format?
Human: Temp and precip, summer 2025, Zarr format.
LLM:   Got it. Here's the plan:
       1. Set up project with xarray/zarr deps
       2. Implement downloader with chunking
       3. Add CLI interface
       Ready for Step 1?
Human: Yes, let's go.
```

### Full Session (Team/Complex Project)

```
[TURN 1] Human: I want to build an Airflow DAG for ERA5 downloads.
         LLM:   Questions: variables? schedule? storage location?
         
[TURN 2] Human: [answers questions]
         LLM:   Here's the plan: 1. Setup 2. Loader 3. DAG 4. Tests
         
[TURN 3] Human: Create directory structure.
         LLM:   [outputs file tree + pyproject.toml]
         
[TURN 4] Human: Implement Step 2 (loader).
         LLM:   [code with type hints, docstrings]
         
[TURN 5] Human: I added soil_temp. Check for issues.
         LLM:   "Needs .sel(level=1) for the level dimension"
         
[TURN 6] Human: Provide complete file.
         LLM:   [full working implementation]
```

---

## Anti-Patterns to Avoid

### ❌ Skipping the Plan
```
Bad:  "Build me an ETL pipeline"
Good: [clarify requirements] → [get task list] → "Let's implement Step 1"
```

### ❌ Asking for Everything at Once
```
Bad:  "Generate the entire project with all features"
Good: "Let's implement Step 3.2: lapse rate correction"
```

### ❌ Not Providing Context
```
Bad:  "Add a function to process the data"
Good: "Add a function to process ERA5 data. 
       Here's the existing DataLoader interface: [code]
       It should output: [schema]
       Constraints: [requirements]"
```

### ❌ Never Validating
```
Bad:  Implement → Ship
Good: Implement → "Check for issues" → Fix → Test → Ship
```

### ❌ Huge Commits
```
Bad:  One commit with entire feature
Good: Commit after each task (1.1, 1.2, 1.3...)
```

---

## Prompt Templates Reference

### Phase 1: Requirements
```
I want to build [PROJECT].
Ask me questions until requirements are clear.
```

### Phase 2: Planning
```
Let's plan this out.
Break into numbered steps with complexity estimates.
```

### Phase 3: Structure
```
Create Python project structure (src/ layout) for this.
Include pyproject.toml, tests/, and skeleton files.
```

### Phase 4: Implementation
```
Implement Step [N]: [DESCRIPTION]
Context: [EXISTING CODE/INTERFACES]
Include: type hints, docstrings, error handling, tests.
```

### Phase 5: Review
```
Review this codebase for issues.
Check: consistency, error handling, performance, docs, tests.
```

---

## Key Takeaways

1. **Clarify First** — Don't code until requirements are clear (conversation or spec.md)
2. **Plan the Work** — Break into numbered, trackable tasks (conversation or plan.md)
3. **Structure Matters** — Use src/ layout from the start
4. **Small Steps** — One task per prompt, one commit per task
5. **Always Validate** — "Check for issues" after every change
6. **Context is King** — Show AI the interfaces, constraints, and related code
7. **Challenge Assumptions** — "Why this approach?" "What are the gaps?"
