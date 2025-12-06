---
description: Execute a single atomic commit from a story's commit plan following TDD workflow and quality gates
allowed-tools: Read(*), Edit(*), Write(*), Bash(*), Grep(*), Glob(*)
argument-hint: <story-id> <commit-number>
---

Execute a single atomic commit from a story's commit plan following TDD workflow and quality gates.

## Usage

```
/atomic-commit <story-id> <commit-number>
```

**Examples:**
- `/atomic-commit 1.1 1` - Execute commit 1 from story 1.1
- `/atomic-commit 2.3 5` - Execute commit 5 from story 2.3

## Parameters

- `<story-id>`: Story identifier (e.g., `1.1`, `2.3`)
- `<commit-number>`: Commit number from the atomic commit plan (1-indexed)

## File Resolution

The command will automatically locate:
- **Atomic Commit Plan:** `docs/stories/{story-id}-atomic-commit-plan.md`
- **Test Design:** `docs/qa/assessments/{story-id}-test-design-*.md`
- **Story File:** `docs/stories/{story-id}.story.md`

## Execution Workflow

### Phase 1: Planning & Context Gathering
1. Read the atomic commit plan and locate the specified commit section
2. Read the test design to understand test requirements
3. Extract commit details:
   - Conventional commit message
   - Test IDs to implement/verify
   - Acceptance criteria covered
   - Priority level
   - Setup steps
   - Verification steps

### Phase 2: Codebase Reconnaissance
4. **Before implementing**, understand the implementation context:
   - Use the Task tool with `subagent_type=Explore` to gather codebase context
   - Identify target files that need to be created or modified
   - Locate entry points and understand existing patterns
   - Review relevant layer architecture (Data → Signal → Portfolio → Backtest → Utils)
   - Understand existing similar implementations for consistency
   - Note any architectural constraints (e.g., pure functions in Signal/Portfolio layers)

   **Prompt for Explore agent:**
   ```
   Review the codebase to understand the implementation context for this commit.

   **Required Documents to Review:**
   - Atomic Commit Plan: docs/stories/{story-id}-atomic-commit-plan.md
   - Test Design: docs/qa/assessments/{story-id}-test-design-*.md
   - Story File: docs/stories/{story-id}.story.md
   - Architecture Docs: docs/architecture/ (especially ADR-001, ADR-002, ADR-004, ADR-010)
   - Project Instructions: CLAUDE.md (layered architecture, pure functions, test organization)

   **Commit Context:**
   - Commit message: {conventional_commit_message}
   - Test IDs to implement: {test_ids}
   - Acceptance Criteria covered: {acceptance_criteria}
   - Setup steps from commit plan: {setup_steps}
   - Files/paths mentioned in setup: {files_from_setup}

   **Investigation Tasks:**
   1. Review the commit plan's setup steps and identify target files for creation/modification
   2. Read the test design document to understand test requirements and expected behavior
   3. Check story file's Dev Notes and Testing sections for implementation guidance
   4. Find similar existing implementations in the codebase for pattern consistency
   5. Identify which architecture layer(s) this commit affects (Data/Signal/Portfolio/Backtest/Utils)
   6. Review relevant ADRs for architectural constraints (especially pure functions in Signal/Portfolio)
   7. Locate entry points and integration patterns in existing code
   8. Find any existing test fixtures or utilities that can be reused

   **Deliverable - Provide a brief summary including:**
   - Document references: Key sections from atomic plan, test design, story file, and ADRs
   - Target files: What files need to be created or modified (with exact paths)
   - Implementation patterns: Existing code patterns to follow for consistency
   - Architectural constraints: Layer-specific rules (e.g., pure functions, no I/O, df.copy())
   - Dependencies: What existing modules/functions to import or build upon
   - Test patterns: Existing test fixtures, markers, and organization to follow
   - Integration points: How this commit connects with existing code
   ```

### Phase 3: Implementation (Flexible TDD)
5. Implement the setup steps from the commit plan
   - Create/modify files as needed
   - Write test code if tests don't exist (see **Test Traceability Format** below)
   - Implement production code
   - Update configuration files
6. Follow the setup instructions precisely as documented in the commit plan

### Phase 4: Verification
7. Run commit-specific tests to verify implementation:
   - Execute the specific test IDs mentioned in the commit plan
   - Ensure all commit-specific tests pass
8. Verify test traceability and completeness (see **Test Traceability Verification** below)
9. Verify setup-specific commands from the commit plan (e.g., `uv --version`, `mypy src/`)

### Phase 5: Quality Gates
10. Run full quality gate validation (see **Quality Gate Requirements** below)
11. If any quality gate fails, STOP and report the failure. Do NOT commit.

### Phase 6: Story File Update
12. Update the story file (`docs/stories/{story-id}.story.md`) with completion details (see **Story File Update Format** below):
    - Check off completed task boxes
    - Update Agent Model Used (if needed)
    - Add Debug Log References (if applicable)
    - Add Completion Notes
    - Add modified files to File List

**IMPORTANT:**
- Do NOT modify the "Status" field
- Do NOT modify any other sections (Story, Acceptance Criteria, Dev Notes, Testing, Change Log, QA Results)
- Only update the four subsections listed in the "Dev Agent Record" section
- Only check off tasks that are actually complete based on this commit's work

### Phase 7: Git Commit
13. Stage all relevant files (tests + implementation + config + story file)
14. Create commit using the conventional commit message from the plan
15. Append Claude Code attribution:
    ```
    🤖 Generated with [Claude Code](https://claude.com/claude-code)

    Co-Authored-By: Claude <noreply@anthropic.com>
    ```
16. Verify commit was created successfully with `git log -1 --oneline`

## Quality Gate Requirements

All of the following must pass before committing:
- ✅ Commit-specific tests pass
- ✅ `uv run mypy src/ tests/` returns exit code 0
- ✅ `uv run ruff check .` returns exit code 0
- ✅ `uv run ruff format .` completes successfully
- ✅ `uv run pytest tests/ -v` - all tests pass (full suite)

## Test Traceability Format

All test code must include traceability markers to link back to the test design document.

### Required Elements

1. **Test ID** - First line of docstring (e.g., `"""Test ID: 1.1-INT-001"""`)
2. **Test Description** - Match the description from test design document
3. **Test Design Reference** - Link to specific test design section
4. **Test Steps** - Document the test steps (from test design) in docstring or comments
5. **Clear Function Name** - Descriptive name that indicates test purpose

### Format Template

```python
def test_descriptive_name():
    """Test ID: {story-id}-{test-type}-{number}

    {Brief description matching test design}

    Ref: docs/qa/assessments/{story-id}-test-design-*.md#{section}

    Steps:
    1. {Step 1 from test design}
    2. {Step 2 from test design}
    3. {Step 3 from test design}

    Expected: {Expected result from test design}
    """
    # Test implementation
```

### Example 1: Integration Test

```python
def test_uv_installed():
    """Test ID: 1.1-INT-001

    Verify UV package manager is installed and accessible on the system.

    Ref: docs/qa/assessments/1.1-test-design-20251117.md#integration-tests

    Steps:
    1. Execute `uv --version` command
    2. Verify command returns exit code 0
    3. Verify output contains version number in format X.Y.Z

    Expected: UV version is displayed (e.g., "uv 0.5.0") with exit code 0
    """
    result = subprocess.run(
        ["uv", "--version"],
        capture_output=True,
        text=True
    )

    # Verify exit code (Step 2)
    assert result.returncode == 0, "uv command should execute successfully"

    # Verify version format (Step 3)
    version_pattern = r"uv \d+\.\d+\.\d+"
    assert re.search(version_pattern, result.stdout), \
        f"Expected version format 'uv X.Y.Z', got: {result.stdout}"
```

### Example 2: Unit Test

```python
def test_calculate_momentum_score():
    """Test ID: 2.3-UNIT-005

    Verify momentum score calculation for a single ETF returns correct value.

    Ref: docs/qa/assessments/2.3-test-design-20251118.md#unit-tests

    Steps:
    1. Create sample price data with known momentum pattern
    2. Call calculate_momentum_score() with sample data
    3. Verify returned score matches expected calculation
    4. Verify score is within valid range [0, 100]

    Expected: Momentum score of 75.5 for upward trending test data
    """
    # Step 1: Create sample data with upward trend
    price_data = pd.Series([100, 105, 110, 115, 120])

    # Step 2: Calculate momentum score
    score = calculate_momentum_score(price_data)

    # Step 3: Verify expected value
    assert score == pytest.approx(75.5, rel=0.01), \
        f"Expected momentum score 75.5, got {score}"

    # Step 4: Verify valid range
    assert 0 <= score <= 100, \
        f"Momentum score {score} outside valid range [0, 100]"
```

### Test Traceability Verification

Before committing, verify each test includes:
- [ ] Test ID in first line of docstring
- [ ] Description matches test design document
- [ ] Reference to test design document with section anchor
- [ ] Test steps documented (numbered list in docstring)
- [ ] Expected results documented
- [ ] Implementation follows test design specifications
- [ ] Comments reference specific steps where helpful

Cross-reference each implemented test against the test design document:
- Confirm test ID is present in test docstring (first line)
- Verify test implementation matches test design specifications:
  * Test description aligns with test design
  * All test design steps are implemented
  * Expected results match test design assertions
- Ensure all test IDs mentioned in commit plan have corresponding test implementations
- Confirm no test design requirements were missed

## Story File Update Format

### Tasks/Subtasks Checkboxes

Only check off tasks/subtasks that were **fully completed** in this commit:
- Match task descriptions to the work done (don't check off unrelated tasks)
- A task may span multiple commits - only check it off when it's fully done
- Format change: `- [ ] Task description` → `- [x] Task description`

**Example:**
```markdown
- [x] Task 1: Initialize UV project and configure pyproject.toml (AC: 1, 2, 3)
  - [x] Verify UV is installed (check with `uv --version`)
  - [x] Run `uv init` to create initial `pyproject.toml`
  - [x] Set Python version to 3.13.1 in `pyproject.toml`
  - [ ] Add core dependencies: pandas 2.2.0, numpy 1.26.3, scipy 1.12.0...
```

### Agent Model Used

If currently "_Not yet implemented_", replace with the model name:

```markdown
### Agent Model Used

claude-sonnet-4-5-20250929
```

### Debug Log References

Add entries for issues encountered, decisions made, workarounds, or deviations:

**Format:** `- [Commit {N}] {Description of issue/decision}: {Reasoning/resolution}`

```markdown
### Debug Log References

- [Commit 1] UV version incompatibility: System had UV 0.4.9, needed 0.5.0+. Resolution: Updated UV via pip
- [Commit 3] Dependency conflict between numpy 1.26.3 and scipy 1.12.0: Relaxed numpy constraint to >=1.26.0
```

### Completion Notes List

Add notes about what was implemented, observations, and related links:

**Format:** `- [Commit {N}] {Note description}`

```markdown
### Completion Notes List

- [Commit 1] Successfully initialized UV project with Python 3.13.1 requirement
- [Commit 2] All dependencies declared following exact versions from tech stack document
- [Commit 3] Virtual environment created successfully, all quality gates passing
```

### File List

Add all files created or modified in this commit:

**Format:** `- {file_path} - {brief description}`

```markdown
### File List

- pyproject.toml - UV project configuration with Python 3.13+ and all dependencies
- mypy.ini - Type checker configuration in strict mode
- pytest.ini - Test framework configuration with coverage reporting
- ruff.toml - Linter and formatter configuration
- .gitignore - Version control exclusions
- src/__init__.py - Python package marker
- tests/__init__.py - Test package marker
- tests/test_environment.py - Environment health check tests
```

## Success Criteria

The command completes successfully when:
1. All setup steps from the commit plan are executed
2. All commit-specific tests pass
3. All quality gates pass
4. Story file is updated with completion details
5. Git commit is created with correct conventional commit message
6. `git status` shows a clean working tree (or only files for future commits)

## Error Handling

If any phase fails:
- **Phase 1-2 (Planning/Implementation):** Report the error and provide context about what failed
- **Phase 3 (Verification):** Show test failures and stop before quality gates
- **Phase 4 (Quality Gates):** Show which gate failed (mypy/ruff/pytest) and stop before story update
- **Phase 5 (Story File Update):** Report parsing/editing errors and stop before commit
- **Phase 6 (Git Commit):** Report git errors and provide guidance

DO NOT create a commit if any verification or quality gate fails.

## Output Format

Provide clear status updates for each phase:

```
📋 Phase 1: Planning & Context Gathering
  ✅ Located atomic commit plan: docs/stories/1.1-atomic-commit-plan.md
  ✅ Located test design: docs/qa/assessments/1.1-test-design-20251117.md
  ✅ Extracted commit 1 details:
     - Message: build(setup): initialize uv project with python 3.13+
     - Tests: 1.1-INT-001, 1.1-INT-002, 1.1-INT-003
     - ACs: 1
     - Priority: P0

🔍 Phase 1.5: Codebase Reconnaissance
  ✅ Explored codebase for implementation context
  ✅ Identified target files and patterns
  ✅ Reviewed architectural constraints
  ✅ Located similar implementations for consistency

🛠️ Phase 2: Implementation
  ✅ Created pyproject.toml with Python 3.13+
  ✅ Wrote tests/test_environment.py with test_uv_installed
  ...

🧪 Phase 3: Verification
  ✅ All commit-specific tests pass (3/3)
  ✅ Setup verification commands pass

🎯 Phase 4: Quality Gates
  ✅ mypy validation passed
  ✅ ruff check passed
  ✅ ruff format completed
  ✅ Full test suite passed (3/3 tests)

📝 Phase 5: Story File Update
  ✅ Checked off 3 completed subtasks
  ✅ Updated Agent Model Used: claude-sonnet-4-5-20250929
  ✅ Added 0 debug log entries
  ✅ Added 1 completion note
  ✅ Added 2 files to file list

📦 Phase 6: Git Commit
  ✅ Staged 3 files (including story file)
  ✅ Created commit: abc123d build(setup): initialize uv project with python 3.13+
  ✅ Verification: git log shows new commit

✨ Atomic commit 1 completed successfully!
```

## Notes for Orchestrators

When using this command in an orchestration workflow:
- Execute commits sequentially in order (commit 1, then 2, then 3, etc.)
- Do not proceed to next commit if current commit fails
- Each commit should leave the repository in a working, tested state
- Commits may have dependencies on previous commits (check the plan's dependency graph)
- Story file updates accumulate across commits - don't remove previous entries

## Traceability

Each commit includes traceability to:
- Test IDs (e.g., 1.1-INT-001)
- Acceptance Criteria (e.g., AC 1)
- Priority level (P0, P1, P2)
- Atomic commit plan document
