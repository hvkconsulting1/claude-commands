---
description: Run post-story technical debt assessment and update tracking documents
allowed-tools: Read(*), Edit(*), Write(*), Bash(*), Grep(*), Glob(*)
argument-hint: <story-id>
---

# Assess Story - Post-Story Technical Debt Assessment

**Role:** You are the **Technical Debt Auditor** performing systematic post-story assessment to identify and track technical debt, architecture drift, and quality erosion in an AI-agent-first development environment.

**Usage:** `/assess-story <story-id>`

**Example:** `/assess-story 1.2`

---

## Document Architecture (No Duplication)

This workflow follows a **single source of truth** pattern to eliminate duplication:

```
┌─────────────────────────────────────────────────────────────┐
│ SINGLE SOURCE OF TRUTH (Detailed)                          │
│ docs/qa/assessments/{story-id}-debt-assessment-{date}.md    │
│                                                             │
│ • All findings, analysis, scores                           │
│ • All recommendations                                      │
│ • References type-ignore-registry for type ignores        │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   │ Referenced by (no duplication)
                   │
       ┌───────────┴────────────┐
       │                        │
       ▼                        ▼
┌─────────────────┐    ┌──────────────────────┐
│ docs/           │    │ docs/                │
│ tech-debt.md    │    │ type-ignore-         │
│                 │    │ registry.md          │
│ • Story index   │    │                      │
│ • Links to      │    │ • Active type        │
│   assessments   │    │   ignores only       │
│ • Active cross- │    │ • Specialized        │
│   story debt    │    │   tracking           │
└─────────────────┘    └──────────────────────┘
       │
       │ Summarized by
       │
       ▼
┌─────────────────┐
│ Console Output  │
│                 │
│ • Key metrics   │
│ • Pointers to   │
│   detailed docs │
└─────────────────┘
```

**Key Principle:** Information lives in ONE place. Other documents reference it, not duplicate it.

**What This Eliminates:**
- ❌ Same findings in 3 places
- ❌ Same recommendations in 3 places
- ❌ Type ignores duplicated across documents
- ✅ Single source, multiple views

---

## Workflow Overview

This command performs a comprehensive technical debt assessment after a story is completed:

1. **Analysis Phase** - Run automated checks for technical debt indicators
2. **Document Creation** - Create or update tracking documents (following no-duplication architecture)
3. **Report Generation** - Generate story-specific assessment report (single source of truth)
4. **Summary Display** - Show key metrics and point to detailed assessment

---

## Context: Why This Matters

In an AI-agent-first workflow with parallel atomic commits:
- Multiple agents work independently without seeing each other's solutions
- Agents optimize locally, requiring systematic global reviews
- Technical debt accumulates at story boundaries
- The story boundary is the perfect checkpoint for assessment

---

## Phase 1: Analysis & Data Collection

### Step 1.1: Determine Git Range

Identify the git commit range for this story using the atomic-plan-assessment file as the boundary (the last planning artifact before implementation):

```bash
# Find the atomic-plan-assessment file for this story
ASSESSMENT_FILE=$(ls docs/qa/assessments/{story-id}-atomic-plan-assessment-*.md 2>/dev/null | head -1)

# Get the commit where this file was added
PLANNING_COMMIT=$(git log --diff-filter=A --format="%H" -1 -- "$ASSESSMENT_FILE")

# Find current HEAD
HEAD_COMMIT=$(git rev-parse HEAD)
```

**Output:** Store `PLANNING_COMMIT` and `HEAD_COMMIT` for diff analysis

**Rationale:** The atomic-plan-assessment file is the final planning document before implementation begins, making it the logical boundary for measuring technical debt introduced during story execution.

### Step 1.2: Run Technical Debt Checks

Execute the following checks in parallel and collect results:

#### Check 1: Type Ignore Comments Added

```bash
# Count type: ignore comments added (only added lines, not removed)
git diff ${PLANNING_COMMIT} HEAD --unified=0 | grep -c "^\+.*type: ignore" || echo "0"

# List locations
git diff ${PLANNING_COMMIT} HEAD --unified=0 | grep "^\+.*type: ignore"
```

**Output:** Count and list of new `type: ignore` comments

#### Check 2: TODOs/FIXMEs Added

```bash
# Count TODOs and FIXMEs added
git diff ${PLANNING_COMMIT} HEAD --unified=0 | grep -E "^\+.*TODO|^\+.*FIXME" | wc -l

# List them
git diff ${PLANNING_COMMIT} HEAD --unified=0 | grep -E "^\+.*TODO|^\+.*FIXME"
```

**Output:** Count and list of new TODO/FIXME comments

#### Check 3: New Fixtures Analysis

```bash
# Find new fixtures in story conftest
find tests/stories/{story-id} -name conftest.py 2>/dev/null || echo "No story conftest found"

# If found, extract fixture definitions
if [ -f tests/stories/{story-id}/conftest.py ]; then
    grep -E "^def |^@pytest.fixture" tests/stories/{story-id}/conftest.py
fi

# Compare with global fixtures
grep -E "^def |^@pytest.fixture" tests/conftest.py
```

**Output:** List of new story-level fixtures and potential promotion candidates

#### Check 4: Architecture Boundary Violations

```bash
# Check for layer violations in Signal layer
grep -r "from momo.backtest\|from momo.portfolio" src/momo/signals/ 2>/dev/null || echo "✓ No violations in signals/"

# Check for layer violations in Portfolio layer
grep -r "from momo.backtest" src/momo/portfolio/ 2>/dev/null || echo "✓ No violations in portfolio/"

# Check for direct norgatedata imports outside bridge
grep -r "import norgatedata\|from norgatedata" src/momo/ | grep -v "src/momo/data/bridge.py" || echo "✓ No violations in norgatedata usage"

# Check for mutations in pure function layers (missing .copy())
grep -r "def calculate_\|def rank_\|def construct_" src/momo/signals/ src/momo/portfolio/ -A 10 | grep -v "\.copy()" | grep -E "^\s*[a-z_]+\[" || echo "✓ No obvious mutations detected"
```

**Output:** List of architecture violations or confirmation of compliance

#### Check 5: Test File Naming Compliance

```bash
# Find test files that don't follow convention
find tests/stories/{story-id} -name "test_*.py" 2>/dev/null | grep -v -E "test_[0-9]+\.[0-9]+_(unit|integration|e2e)_[0-9]+\.py" || echo "✓ All test files follow naming convention"
```

**Output:** List of non-compliant test file names or confirmation of compliance

#### Check 6: Test Marker Consistency

```bash
# Find tests missing priority markers
for test_file in tests/stories/{story-id}/*/test_*.py; do
    if [ -f "$test_file" ]; then
        if ! grep -q "@pytest.mark.p[012]" "$test_file"; then
            echo "Missing priority marker: $test_file"
        fi
    fi
done

# Find tests missing level markers
for test_file in tests/stories/{story-id}/*/test_*.py; do
    if [ -f "$test_file" ]; then
        if ! grep -q "@pytest.mark.\(unit\|integration\|e2e\)" "$test_file"; then
            echo "Missing level marker: $test_file"
        fi
    fi
done
```

**Output:** List of tests missing markers

#### Check 7: Code Duplication Patterns

```bash
# Look for similar function names across files (potential duplication)
find src/momo -name "*.py" -exec grep -h "^def " {} \; | sort | uniq -d | head -20
```

**Output:** List of potentially duplicated function names

#### Check 8: Test Coverage Analysis

```bash
# Run full test suite to check for regressions and get coverage
uv run pytest --cov=src/momo --cov-report=term-missing --quiet 2>&1 | tail -20
```

**Output:** Full test suite results and overall coverage statistics (detects regressions)

---

## Phase 2: Document Creation/Update

### Step 2.1: Create or Update docs/tech-debt.md

**Architecture Principle:** `tech-debt.md` is a **lightweight index and active debt tracker**, NOT a duplicate of assessment details. It tracks cross-story consolidation opportunities and provides a dashboard view.

**First Run:** If `docs/tech-debt.md` doesn't exist, create it with this template:

```markdown
# Technical Debt Tracker

**Last Updated:** {current-date}

**Purpose:** Track active cross-story technical debt and consolidation opportunities. This is an index/dashboard - see individual story assessments for detailed findings.

---

## Active Consolidation Opportunities

### Fixture Consolidation
<!-- Fixtures that should be promoted to global scope -->

**None currently tracked**

### Code Duplication
<!-- Patterns repeated across stories that should be extracted -->

**None currently tracked**

### Architecture Improvements
<!-- Design issues or boundary violations requiring attention -->

**None currently tracked**

---

## Story Assessment Index

| Story | Date | Score | Grade | Key Issues | Report |
|-------|------|-------|-------|------------|--------|

**Note:** Click report links for detailed findings, analysis, and recommendations.

---
```

**Update Existing:** If `docs/tech-debt.md` exists:

1. **Update "Active Consolidation Opportunities"** section:
   - Add new items discovered in this story that affect multiple locations
   - Update existing items if this story added to them
   - **Do NOT duplicate detailed findings here** - those go in the assessment report

2. **Add entry to "Story Assessment Index"** table:

```markdown
| {story-id} | {date} | {score}/100 | {grade} | {1-2 word summary} | [Assessment](qa/assessments/{story-id}-debt-assessment-{date}.md) |
```

**Example entries for Active Opportunities:**

```markdown
### Fixture Consolidation

- **`sample_price_data`** - Used in stories 1.2, 1.3, 1.4 story conftest files. Should promote to `tests/conftest.py`. [Story 1.4 Assessment](qa/assessments/1.4-debt-assessment-20250115.md#fixtures)

### Code Duplication

- **Momentum calculation helpers** - Similar logic in `signals/momentum.py` and `signals/ranking.py`. Extract to shared utility. [Story 2.1 Assessment](qa/assessments/2.1-debt-assessment-20250120.md#duplication)
```

### Step 2.2: Create or Update docs/type-ignore-registry.md

**Architecture Principle:** `type-ignore-registry.md` is the **single source of truth** for all type ignore comments. Assessment reports **reference** this registry, not duplicate it.

**First Run:** If `docs/type-ignore-registry.md` doesn't exist, create it with this template:

```markdown
# Type Ignore Registry

This document tracks all `type: ignore` comments in the codebase with justification and removal plans.

**Last Updated:** {current-date}

**Purpose:** Every `type: ignore` comment represents a type safety gap. This registry ensures each one is justified, tracked, and has a plan for resolution or permanent acceptance.

**Policy:**
- All `type: ignore` comments must be registered here
- Each entry must have a justification
- Each entry must have a removal plan or permanent acceptance reason
- Entries should be resolved or permanently accepted within 2-3 stories

---

## Active Type Ignores

<!-- Format: Location | Added | Reason | Plan | Status -->

| Location | Added | Reason | Plan | Status |
|----------|-------|--------|------|--------|

---

## Story-by-Story Additions

<!-- Reverse chronological order - newest first -->

```

**Update Existing:** If `docs/type-ignore-registry.md` exists, read it and append new entries.

**Story Entry Format:**

```markdown
### Story {story-id} - {date}

{list each type: ignore added}

**Entries Added to Registry:**

| Location | Reason | Plan | Status |
|----------|--------|------|--------|
| {file}:{line} | {why it's needed} | {how to remove it} | 🔴 New |

---
```

**For each `type: ignore` found:**
1. Extract file path, line number, and surrounding context
2. Add entry to "Active Type Ignores" table
3. Add entry to story-specific section
4. Mark status as 🔴 New

---

## Phase 3: Generate Story Assessment Report

### Step 3.1: Create Story-Specific Assessment File

**Architecture Principle:** This is the **single source of truth** for detailed story findings. Other documents reference this, not duplicate it.

Create `docs/qa/assessments/{story-id}-debt-assessment-{YYYYMMDD}.md` with detailed findings:

```markdown
# Story {story-id} - Technical Debt Assessment

**Assessment Date:** {current-date}
**Story Status:** {status from story file}
**Commit Range:** {PLANNING_COMMIT}..{HEAD_COMMIT}
**Assessor:** Claude Code (claude-sonnet-4-5-20250929)

---

## Executive Summary

Story {story-id} introduced **{overall-rating}** technical debt:
- {brief summary of findings}
- {overall health assessment}

**Overall Debt Score:** {calculate 0-100, where 100 is zero debt}

---

## Detailed Findings

### 1. Type Safety ({score}/100)

**Type Ignores Added:** {count}

{if count > 0:}
See [Type Ignore Registry](../../type-ignore-registry.md#story-{story-id}) for detailed tracking.

**Locations Added:**
- {file}:{line} - {brief context}
- {file}:{line} - {brief context}

{else:}
None - excellent type safety maintained.

**Assessment:** {evaluation}

**Action Required:** {specific tasks}

---

### 2. Code Organization ({score}/100)

**TODOs/FIXMEs Added:** {count}

{detailed list or "None - no unresolved work items"}

**Assessment:** {evaluation}

**Action Required:** {specific tasks}

---

### 3. Test Infrastructure ({score}/100)

**New Fixtures:** {count}
**Naming Compliance:** {pass/fail}
**Marker Compliance:** {pass/fail}

{detailed findings}

**Fixtures Needing Promotion:**
1. {fixture} - {rationale}

**Assessment:** {evaluation}

**Action Required:** {specific tasks}

---

### 4. Architecture Compliance ({score}/100)

**Violations Detected:** {count}

{detailed list or "✓ Full compliance with layered architecture"}

**Assessment:** {evaluation}

**Action Required:** {specific tasks}

---

### 5. Code Duplication ({score}/100)

**Duplicate Patterns:** {count}

{detailed findings}

**Assessment:** {evaluation}

**Action Required:** {specific tasks}

---

### 6. Test Coverage ({score}/100)

**Full Test Suite Status:** {all passing / X failures}
**Overall Coverage:** {percentage}%
**Coverage Change:** {+/-X}%

{detailed coverage report}

**Regressions Detected:** {yes/no - list any failing tests outside story scope}

**Assessment:** {evaluation}

---

## Consolidation Opportunities

### High Priority
1. {specific opportunity with impact}
2. {specific opportunity with impact}

### Medium Priority
1. {specific opportunity with impact}

### Low Priority
1. {specific opportunity with impact}

---

## Recommended Actions

### Do Now (Before Next Story)
- [ ] {specific task with acceptance criteria}
- [ ] {specific task with acceptance criteria}

### Do Next Story
- [ ] {specific task with acceptance criteria}

### Backlog (Future Cleanup)
- [ ] {specific task with acceptance criteria}

---

## Metrics Summary

| Metric | Value | Trend | Target |
|--------|-------|-------|--------|
| Type Ignores | {count} | {↑/↓/→} | 0 |
| TODOs | {count} | {↑/↓/→} | <3 per story |
| Architecture Violations | {count} | {↑/↓/→} | 0 |
| Test Markers Missing | {count} | {↑/↓/→} | 0 |
| Fixtures Needing Promotion | {count} | {↑/↓/→} | 0 |
| Code Duplication Instances | {count} | {↑/↓/→} | <2 per story |

---

## Comparison to Previous Story

{if previous assessment exists, compare key metrics}

**Trends:**
- {improving/stable/degrading patterns}

---

## Automated Check Commands

Run these commands to verify issues:

```bash
# Type ignores (added only)
git diff {PLANNING_COMMIT} HEAD --unified=0 | grep "^\+.*type: ignore"

# TODOs (added only)
git diff {PLANNING_COMMIT} HEAD --unified=0 | grep -E "^\+.*TODO|^\+.*FIXME"

# Architecture violations
grep -r "from momo.backtest" src/momo/signals/ src/momo/portfolio/

# Test markers
find tests/stories/{story-id} -name "test_*.py" -exec grep -L "@pytest.mark.p[012]" {} \;
```

---

## Notes

{any additional observations or context}

---

**Assessment Complete** ✓
```

---

## Phase 4: Generate Summary Report

### Step 4.1: Display Summary to User

Provide a concise summary that **points to the detailed assessment** rather than duplicating it:

```
🔍 Story {story-id} Technical Debt Assessment Complete

📊 Overall Health Score: {score}/100 ({grade})

Key Findings:
  • Full Test Suite: {all passing / X failures} {status-emoji}
  • Regressions: {yes/no} {status-emoji}
  • Type Ignores: {count} {status-emoji}
  • TODOs/FIXMEs: {count} {status-emoji}
  • Architecture: {compliant/violations} {status-emoji}
  • Test Quality: {rating} {status-emoji}
  • Code Duplication: {rating} {status-emoji}

📁 Documents Updated:
  ✓ docs/tech-debt.md (index updated)
  ✓ docs/type-ignore-registry.md (if type ignores found)
  ✓ docs/qa/assessments/{story-id}-debt-assessment-{date}.md

📋 View Details:
  # Full assessment with recommendations
  cat docs/qa/assessments/{story-id}-debt-assessment-{date}.md

  # Dashboard view across all stories
  cat docs/tech-debt.md

  # Type ignore tracking
  cat docs/type-ignore-registry.md

{if regressions detected:}
🚨 REGRESSIONS DETECTED - Tests that previously passed are now failing!
   Review full test output and fix before proceeding to next story.
{endif}

{if score < 70:}
⚠️  Score below 70 - Review recommendations in assessment before next story
{endif}

{if previous story exists:}
📈 Trend: {improving/stable/degrading} compared to Story {prev-id} ({prev-score}/100)
{endif}
```

---

## Scoring Rubric

### Overall Health Score (0-100)

Calculate as weighted average:
- Type Safety: 25%
- Code Organization: 15%
- Test Infrastructure: 20%
- Architecture Compliance: 25%
- Code Duplication: 10%
- Test Coverage: 5%

### Individual Category Scoring

**Type Safety (0-100):**
- 100: 0 type ignores
- 90: 1 type ignore
- 70: 2-3 type ignores
- 50: 4-5 type ignores
- 30: 6-10 type ignores
- 0: 10+ type ignores

**Code Organization (0-100):**
- 100: 0 TODOs
- 90: 1-2 TODOs with clear resolution plan
- 70: 3-4 TODOs
- 50: 5-7 TODOs
- 30: 8-10 TODOs
- 0: 10+ TODOs or any FIXME without issue tracking

**Test Infrastructure (0-100):**
- 100: All tests follow conventions, no fixture duplication
- 90: 1 minor issue (missing marker, etc.)
- 80: 1-2 fixtures should be promoted
- 60: Multiple marker issues or 3+ fixture candidates
- 40: Test naming violations
- 0: Test ID mapping broken

**Architecture Compliance (0-100):**
- 100: Zero violations
- 70: 1 minor violation (acceptable with justification)
- 40: 2-3 violations
- 0: 4+ violations or any critical violation (direct norgatedata import, etc.)

**Code Duplication (0-100):**
- 100: No duplication detected
- 80: 1-2 similar patterns (acceptable)
- 60: 3-4 duplicate patterns
- 40: 5+ duplicate patterns
- 0: Obvious copy-paste across multiple files

**Test Coverage (0-100):**
- Based on story-specific coverage percentage

### Grade Mapping

- 90-100: A (Excellent)
- 80-89: B (Good)
- 70-79: C (Acceptable)
- 60-69: D (Needs Improvement)
- 0-59: F (Significant Issues)

---

## Error Handling

If any check fails:
- Document the failure in the assessment report
- Mark that category as "Unable to Assess"
- Provide manual verification commands
- Continue with remaining checks

---

## Success Criteria

The command completes successfully when:
1. All automated checks execute (with or without findings)
2. `docs/tech-debt.md` is created or updated
3. `docs/type-ignore-registry.md` is created or updated (if type ignores found)
4. Story-specific assessment report is created
5. Summary report is displayed to user
6. All documents are properly formatted and readable

---

## Future Enhancements

Potential additions for future iterations:
- Trend analysis across multiple stories
- Automated fixture promotion suggestions with diffs
- Integration with git blame for accountability
- Technical debt "velocity" tracking
- Automated refactoring task generation
- Integration with project management tools

---

## Notes

- This assessment is advisory, not blocking
- Findings should inform planning for next story
- High-debt stories may need a cleanup story before proceeding
- The assessment documents become input artifacts for subsequent cleanup work
- Run this command immediately after story completion, before starting next story

---

**Remember:** In an AI-agent-first workflow, systematic assessment is critical. Agents optimize locally; you assess globally.
