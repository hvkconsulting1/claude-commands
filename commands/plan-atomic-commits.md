---
description: Generate atomic commit plan from story and test design files
allowed-tools: Read(*), Grep(*), Glob(*), Write(*)
argument-hint: <story-file> <test-design-file>
---

# Plan Atomic Commits

Generate an orchestrated sequence of atomic commits based on Test-Driven Development principles, using story acceptance criteria and test design scenarios.

**Usage:** `/plan-atomic-commits <story-file> <test-design-file>`

**Example:** `/plan-atomic-commits docs/stories/1.1.story.md docs/qa/assessments/1.1-test-design-20251117.md`

> **Important:** This command begins with a codebase discovery phase (Step 0) to ensure generated plans are grounded in the actual project structure, dependencies, test patterns, and architectural constraints. Never generate generic plans without first understanding the specific codebase context.

---

## Step 0: Review Codebase and Environment

**CRITICAL:** Before planning commits, you MUST understand the actual project context. Do NOT proceed with generic plans.

### Required Discovery Tasks

1. **Project Structure Analysis**
   - Use Glob to explore the directory structure
   - Identify existing layers: `src/momo/{data,signals,portfolio,backtest,utils}/`
   - Check which modules already exist and which need to be created
   - Review `tests/stories/` to understand test organization patterns

2. **Configuration Review**
   - Read `pyproject.toml` to understand:
     - Python version requirements
     - Existing dependencies and their versions
     - Tool configurations (pytest, mypy, ruff)
     - Project metadata and build settings
   - Read `pytest.ini` or pyproject.toml `[tool.pytest.ini_options]` for test configuration
   - Check for `.mypy.ini` or mypy configuration in pyproject.toml

3. **Testing Patterns**
   - Review existing test files in `tests/stories/` to understand:
     - Test file naming conventions (should match story-based pattern)
     - Fixture patterns used in `tests/conftest.py`
     - Story-level fixture patterns (e.g., `tests/stories/1.1/conftest.py`)
     - How markers are applied (@pytest.mark.p0, @pytest.mark.unit, etc.)
   - Identify existing test utilities or helpers

4. **Architectural Constraints**
   - Read `CLAUDE.md` to understand:
     - Layer dependencies and data flow rules
     - Pure function requirements for Signal/Portfolio layers
     - Windows Python bridge patterns for Norgate Data
     - Development workflow and conventions
   - Check for ADR (Architecture Decision Records) files in `docs/adr/`

5. **Environment and Tooling**
   - Verify which development commands are configured (from CLAUDE.md)
   - Understand the test execution patterns (pytest markers, coverage, etc.)
   - Check if there are any CI/CD configurations

6. **Existing Implementations (if any)**
   - Search for similar stories that have been implemented
   - Review their test organization and implementation patterns
   - Look for reusable fixtures or utilities

### Output from Discovery

After completing discovery, you should have clear answers to:
- What directories and files already exist?
- What dependencies are already configured?
- What testing patterns are established?
- What architectural constraints must be followed?
- What tools and commands are available?
- Are there similar implementations to reference?

**Use this context to create realistic, actionable commit plans that fit the actual codebase.**

---

## Core Principles

Each atomic commit should be:
- **Atomic**: Can be built, tested, and reviewed independently
- **Single Responsibility**: Addresses one tool/config/component
- **Traceable**: Test IDs and AC numbers provide clear traceability to requirements
- **Test-Covered**: Each commit includes both tests and implementation
- **Reviewable**: Small, focused commits are easier to review
- **Revertable**: Each commit is a safe rollback point
- **Working State**: Leaves system in testable/functional state

## Why Atomic Commits?

The primary goals of atomic commits are:
- **Context Management**: Keep each commit small enough to review and implement within reasonable context windows
- **Cognitive Load**: One clear purpose per commit reduces mental overhead
- **Error Recovery**: Easy to identify, review, and revert specific changes
- **Parallel Work**: Multiple commits can be worked on independently

Typical commit size: One logical component/feature with 3-8 tests, ~100-400 lines of changes (tests + implementation combined).

**Anti-Patterns to Avoid:**
- ❌ Commit 1: Write tests (all failing)
- ❌ Commit 2: Implement code (tests now pass)

**Preferred Pattern:**
- ✅ Commit 1: Implement feature X with its test coverage (tests pass)
- ✅ Commit 2: Implement feature Y with its test coverage (tests pass)

---

## Step 1: Parse Input Files

1. **Read the story file** to extract:
   - Acceptance Criteria (ACs) with numbers and descriptions
   - Tasks/subtasks and their relationships
   - Dev notes for context

2. **Read the test design file** to extract:
   - Test scenario IDs (e.g., 1.1-INT-001)
   - Test priorities (P0, P1, P2)
   - Test phases/execution order
   - AC mappings for each test
   - Test descriptions and expected outcomes

---

## Step 2: Group Tests into Atomic Commits

### Grouping Rules

Group tests and implementation by **logical functionality**, not by test-first/implementation-second phases:

1. **Single Component Focus**: All tests + implementation for one function, class, or module → one commit
2. **Feature Cohesion**: Tests + implementation for a tightly coupled feature → one commit
3. **Dependency Ordering**: Foundation components before dependent components
4. **Size Target**: 3-8 tests with implementation, ~100-400 lines total (adjust for complexity)
5. **Independent Verification**: Each commit should be independently testable and functional

**Examples of Logical Units:**
- Schema validation function + its 5 unit tests
- Cache save/load operations + round-trip tests
- Error handling logic + partial failure tests
- Integration workflow + end-to-end validation tests

### Dependency Hierarchy

- Foundation components before dependent components
- Core functionality before advanced features
- Unit-tested components before integration tests that use them
- Simple operations before complex orchestrations

---

## Step 3: Generate Commit Specifications

For each atomic commit, create a specification using this format:

```markdown
## Commit N: <type>(<scope>): <subject>

**Tests:** <comma-separated test IDs>
**ACs:** <comma-separated AC numbers>
**Priority:** <P0/P1/P2 based on highest priority test in group>

**Setup:**
- <Create test files with specific test scenarios>
- <Implement the functionality being tested>
- <Both tests and implementation in this single commit>
- <what commands to run>

**Verification:**
- <how to verify this commit is successful>
- <which tests should pass>
- <expected tool outputs>
```

### Conventional Commit Format

**Types:**
- `feat`: New feature or capability
- `build`: Build system, dependencies, tooling setup
- `test`: Adding or updating tests
- `style`: Code formatting/linting configuration
- `ci`: Continuous integration setup
- `docs`: Documentation changes

**Scopes:** `setup`, `deps`, `tooling`, `structure`, `git`, `env`, `config`

**Subject:**
- Imperative mood ("add", "configure", "create")
- Lowercase, no period
- Max 50 characters
- Clear and descriptive

---

## Step 4: Structure the Output Plan

Create a markdown document with this structure:

```markdown
# Atomic Commit Plan: Story <story-number>

**Total Commits:** N
**Test Coverage:** X/Y tests (Z P0, W P1, V P2)
**AC Coverage:** A/B acceptance criteria

---

[Individual commit specifications using format from Step 3]

---

## Execution Order Summary

1. **Foundation Setup** (Commits X-Y): Brief description
2. **Tool Configuration** (Commits X-Y): Brief description
3. **Project Structure** (Commits X-Y): Brief description
4. **Validation** (Commit X): Brief description

## Implementation Workflow for Each Commit

Each commit is a **logical unit** containing both tests and implementation:

1. **Implement the logical unit**:
   - Write tests that define the expected behavior
   - Implement the functionality to satisfy those tests
   - Order doesn't matter (TDD, test-after, or interleaved) - use what makes sense

2. **Verify**: Run the commit's tests to confirm they pass

3. **Commit**: Stage tests + implementation together with the commit message

**Key Principle:** One commit = one logical unit of functionality with its test coverage.

**Anti-pattern:** Don't create separate commits for "write tests" and "make tests pass" - this artificially splits logical units and doubles the number of commits without adding value.

## Dependencies Between Commits

- List any commit dependencies (e.g., Commit 2 depends on Commit 1)
```

---

## Step 5: Quality Validation

Before writing the output file, verify:

✅ All test scenarios from test design are included
✅ All acceptance criteria are covered
✅ Commits are ordered by dependencies
✅ Each commit has clear verification steps
✅ Commit messages follow conventional commit format
✅ P0 tests are in early commits (fail fast)
✅ Each commit leaves system in working/testable state

**Context Validation (from Step 0):**
✅ File paths reference actual project structure
✅ Dependencies mentioned are available or will be added
✅ Test file paths follow the project's naming convention
✅ Implementation respects architectural constraints (layer dependencies, pure functions)
✅ Commands use actual tooling (uv run pytest, uv run mypy, etc.)
✅ Fixtures reference actual fixture patterns from conftest.py files
✅ Markers match project conventions (@pytest.mark.p0, @pytest.mark.unit)

---

## Step 6: Write Output File

1. **Extract story number** from the story file path:
   - Example: `docs/stories/1.1.story.md` → `1.1`
   - Example: `docs/stories/2.3.story.md` → `2.3`

2. **Write the plan** to `docs/stories/{story-number}-atomic-commit-plan.md`
   - Example: `docs/stories/1.1-atomic-commit-plan.md`

3. **Confirm to user** that the file was created

### Command Scope

This command should:
- ✅ Write the atomic commit plan to the designated file
- ✅ Use the exact markdown format from Step 4
- ✅ Present a brief confirmation message

This command should NOT:
- ❌ Execute any commits
- ❌ Modify any code files
- ❌ Run any implementation commands

**This is a planning command only.** The output file serves as a roadmap for implementation.

---

## Example Commit Specification

The following example shows how Step 0 discoveries inform the commit plan with specific commands, file paths, and configurations from the actual project:

```markdown
## Commit 1: feat(cache): implement cache path generation with validation

**Tests:** 1.3-UNIT-005, 1.3-UNIT-006, 1.3-UNIT-007
**ACs:** 3
**Priority:** P0

**Setup:**
- Create `tests/stories/1.3/unit/test_1_3_unit_005.py` - Test cache path generation consistency
- Create `tests/stories/1.3/unit/test_1_3_unit_006.py` - Test schema validation for required columns
- Create `tests/stories/1.3/unit/test_1_3_unit_007.py` - Test schema validation for correct dtypes
- Create `src/momo/data/cache.py` with:
  - `get_cache_path()` function implementing `{universe}_{start_date}_{end_date}.parquet` pattern
  - `_validate_price_schema()` function checking columns and dtypes
  - Raise `CacheError` on validation failures
- All functions include type hints and docstrings

**Verification:**
- Run: `uv run pytest tests/stories/1.3/unit/test_1_3_unit_00[5-7].py -v` (all pass)
- Run: `uv run mypy src/momo/data/cache.py` (passes)
```
