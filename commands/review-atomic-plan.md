---
description: Validate completeness of planning artifacts before implementation begins
globs: docs/stories/*.story.md, docs/qa/assessments/*.md
argument-hint: <story-id>
---

You are conducting a comprehensive pre-implementation artifact validation for a story that has completed the planning phase (risk assessment, test design, and atomic commit planning).

## Inputs

Parse the command arguments:
- **story-number**: Story ID (e.g., "1.4", "2.3")

## Your Task

Perform a comprehensive validation of all planning artifacts to ensure they are complete, consistent, and ready for implementation.

## Step 1: Locate Required Artifacts

Read the following documents (fail immediately if any are missing):

1. **Story File**: `docs/stories/{story-number}.story.md`
2. **Risk Profile**: `docs/qa/assessments/{story-number}-risk-*.md` (find most recent by date)
3. **Test Design**: `docs/qa/assessments/{story-number}-test-design-*.md` (find most recent by date)
4. **Commit Plan**: `docs/stories/{story-number}-atomic-commit-plan.md`

If any artifact is missing, document which ones are absent and provide their expected paths.

## Step 2: Validate Each Artifact

For each artifact, validate completeness according to the criteria below:

### Story Artifact Validation
- [ ] Story status is "Draft" or "Ready" (not "Completed" or empty)
- [ ] Acceptance criteria present and testable (at least 3 ACs)
- [ ] Tasks/subtasks section exists with breakdown
- [ ] Dev Notes section includes architecture context
- [ ] File locations specified for new/modified code
- [ ] Story has change log with date and version

### Risk Profile Validation
- [ ] Executive summary with total risk count and score
- [ ] At least 1 critical or high risk identified (if score = 0, verify story is trivial)
- [ ] Each risk has: ID, score, category, probability, impact, mitigation
- [ ] Risk-based testing strategy section present
- [ ] Gate recommendation provided (PASS/CONCERNS/FAIL)
- [ ] YAML block for gate file present
- [ ] Risk matrix table present

### Test Design Validation
- [ ] Test strategy overview with total scenario count
- [ ] Test scenarios organized by acceptance criteria
- [ ] Each scenario has: ID, level, priority, justification, risk coverage
- [ ] Test IDs follow naming convention: {story}-{LEVEL}-{SEQ} (e.g., "1.4-UNIT-001")
- [ ] Priority markers present: P0, P1, or P2 for each test
- [ ] Level markers present: unit, integration, or e2e for each test
- [ ] Test pyramid compliance section with percentages
- [ ] Test fixtures section specifying required test data
- [ ] YAML block for gate file present
- [ ] Coverage gap analysis performed

### Commit Plan Validation
- [ ] Total commit count specified (typically 5-15)
- [ ] Each commit has: title, test IDs, ACs, priority
- [ ] Each commit has setup instructions (tests + implementation)
- [ ] Each commit has verification commands (pytest, mypy)
- [ ] Commit dependencies documented
- [ ] Execution order summary present
- [ ] Each commit follows conventional commit format in title

## Step 3: Cross-Artifact Consistency Checks

Validate consistency across artifacts:

### Acceptance Criteria Coverage
- Extract all AC numbers from story (e.g., AC1, AC2, ...)
- Verify each AC is covered by at least one test scenario in test design
- Verify each AC appears in at least one commit in commit plan
- Report any uncovered ACs

### Test Coverage Completeness
- Extract all test IDs from test design (e.g., 1.4-UNIT-001, 1.4-INT-002)
- Verify each test ID appears in commit plan
- Report any tests designed but not assigned to commits

### Risk Coverage Validation
- Extract all critical risks (score ≥ 9) from risk profile
- Verify each critical risk has at least one P0 test covering it
- Extract all high risks (score ≥ 6) from risk profile
- Verify each high risk has at least one P0 or P1 test covering it
- Report any critical/high risks without test coverage

### Priority Alignment
- Count P0 tests in test design
- Verify P0 tests cover critical risks and core functionality
- Verify test pyramid compliance (unit > integration > e2e)

### File Path Consistency
- Extract file paths from story (src/momo/...)
- Verify same paths referenced in commit plan
- Check for path inconsistencies or typos

## Step 3.5: Implementation Readiness Validation

Verify the codebase and environment are ready for implementation:

### Dependency Pre-Check
- [ ] Check if prerequisite stories are merged (look for story dependencies in Dev Notes)
- [ ] Verify all imported modules exist (e.g., if story imports from `src/momo/utils/exceptions.py`, check it exists and has required base classes)
- [ ] Verify Python version requirement met (run `uv run python --version` and compare to `requires-python` in pyproject.toml)
- [ ] Check for external tool dependencies (e.g., NDU for Norgate, specific libraries)

### Test Data Feasibility
- [ ] All fixtures listed in test design are synthetic or have documented data sources
- [ ] No fixtures require live API calls (except in integration tests marked appropriately)
- [ ] Historical data fixtures (e.g., known corporate actions, delisted stocks) have clear data provenance
- [ ] Large dataset fixtures (e.g., 100+ tickers) can be generated or are cached

### Type System Compatibility (if using strict typing)
- [ ] Complex nested types are compatible with project's Python version
- [ ] Check if `from __future__ import annotations` is needed for forward references
- [ ] Verify mypy strict mode compatibility with dataclasses/complex types in plan

### Test Execution Budget
- [ ] Estimate total test runtime: unit tests + integration tests
- [ ] Flag tests expected to take > 5s (e.g., bridge calls, large dataset operations)
- [ ] Verify total test suite within CI timeout (typically 2-10 minutes)
- [ ] Ensure NDU-dependent tests have skip conditions documented

### Commit Size Validation
- [ ] All commits estimated < 500 LOC (ensure truly atomic)
- [ ] Largest commit identified with file/test count estimate
- [ ] No commit combines multiple unrelated features
- [ ] Commit dependencies are minimal (prefer sequential over complex DAG)

**Note**: Flag issues as WARNINGS (not blockers) unless they prevent starting Commit 1. Most issues can be resolved during implementation.

## Step 4: Traceability Matrix

Build a traceability matrix showing:

| AC | Tests Covering AC | Risks Addressed | Commits Implementing |
|----|-------------------|-----------------|----------------------|
| AC1 | 1.4-UNIT-001, ... | DATA-001, ...   | Commit 1, 2, ...    |
| ... | ...               | ...             | ...                 |

Identify any gaps in traceability.

## Step 5: Readiness Assessment

Evaluate readiness for implementation:

### Blockers (Must Fix)
- Missing required artifacts
- Acceptance criteria not covered by tests
- Critical risks without P0 test coverage
- Test IDs in test design not assigned to commits
- Ambiguous or contradictory requirements across artifacts
- **Prerequisite stories not merged** (dependencies not met)
- **Required base classes/modules don't exist** (imports will fail on Commit 1)

### Warnings (Should Fix)
- High risks without test coverage
- Test pyramid ratios outside targets (70/25/5 ±10%)
- Commits without verification commands
- Missing fixture specifications
- Inconsistent file paths
- **Fixtures requiring complex historical data** (may slow down test implementation)
- **Complex nested types without compatibility check** (may cause mypy issues)
- **Test execution time exceeds 5 minutes** (may slow CI feedback)
- **Commits estimated > 500 LOC** (may not be truly atomic)

### Recommendations
- Opportunities to parallelize commits
- Suggested test execution order improvements
- Documentation improvements

## Step 6: Generate Assessment Report

Write a comprehensive assessment report to:
`docs/qa/assessments/{story-number}-atomic-plan-assessment-{YYYYMMDD}.md`

Use today's date for {YYYYMMDD}.

### Report Structure

```markdown
# Atomic Plan Assessment: Story {story-number}

**Date**: {YYYY-MM-DD}
**Reviewer**: Claude Code (Automated Assessment)
**Story**: {story-number} - {story-title}
**Status**: {READY / NEEDS REVISION / BLOCKED}

---

## Executive Summary

- **Artifacts Found**: {count}/4
- **Blockers**: {count}
- **Warnings**: {count}
- **Overall Readiness**: {READY / NEEDS REVISION / BLOCKED}

**Recommendation**: {One-sentence recommendation}

---

## Artifact Validation Results

### Story Artifact
**Location**: `docs/stories/{story-number}.story.md`
**Status**: {PASS / FAIL}

{Checklist results}

**Issues**:
- {List any validation failures}

---

### Risk Profile Artifact
**Location**: `docs/qa/assessments/{story-number}-risk-{date}.md`
**Status**: {PASS / FAIL}

{Checklist results}

**Issues**:
- {List any validation failures}

---

### Test Design Artifact
**Location**: `docs/qa/assessments/{story-number}-test-design-{date}.md`
**Status**: {PASS / FAIL}

{Checklist results}

**Issues**:
- {List any validation failures}

---

### Commit Plan Artifact
**Location**: `docs/stories/{story-number}-atomic-commit-plan.md`
**Status**: {PASS / FAIL}

{Checklist results}

**Issues**:
- {List any validation failures}

---

## Cross-Artifact Consistency Analysis

### Acceptance Criteria Coverage

**Total ACs**: {count}
**ACs Covered by Tests**: {count}
**ACs Covered by Commits**: {count}
**Uncovered ACs**: {list or "None"}

{Details table if gaps exist}

---

### Test Coverage Completeness

**Total Tests Designed**: {count}
**Tests Assigned to Commits**: {count}
**Unassigned Tests**: {list or "None"}

{Details if gaps exist}

---

### Risk Coverage Validation

**Critical Risks (score ≥ 9)**: {count}
**Critical Risks with P0 Coverage**: {count}
**High Risks (score ≥ 6)**: {count}
**High Risks with P0/P1 Coverage**: {count}

**Uncovered Critical Risks**: {list or "None - ✅"}
**Uncovered High Risks**: {list or "None - ✅"}

{Details if gaps exist}

---

### Priority Alignment

**P0 Tests**: {count} ({percentage}%)
**P1 Tests**: {count} ({percentage}%)
**P2 Tests**: {count} ({percentage}%)

**Test Pyramid**:
- Unit: {count} ({percentage}%)
- Integration: {count} ({percentage}%)
- E2E: {count} ({percentage}%)

**Compliance**: {PASS / FAIL} - {explanation}

---

### File Path Consistency

**Files Referenced in Story**: {count}
**Files Referenced in Commit Plan**: {count}
**Inconsistent Paths**: {list or "None - ✅"}

{Details if inconsistencies exist}

---

## Implementation Readiness Validation

### Dependency Pre-Check

**Prerequisite Stories**: {list stories this depends on}
**Dependencies Met**: {✅ / ⚠️ / ❌}

{Check results for prerequisite stories, imported modules, Python version, external tools}

**Issues**:
- {List any dependency issues or "None - ✅"}

---

### Test Data Feasibility

**Total Fixtures Required**: {count from test design}
**Synthetic Fixtures**: {count}
**Cached Data Fixtures**: {count}
**Historical Data Fixtures**: {count}

**Feasibility Assessment**: {PASS / CONCERNS}

**Issues**:
- {List any fixtures that may be difficult to generate or "None - ✅"}

---

### Type System Compatibility

**Complex Types Identified**: {list if any}
**Python Version**: {version from pyproject.toml}
**Mypy Strict Mode**: {enabled/disabled}

**Compatibility**: {✅ PASS / ⚠️ REVIEW NEEDED}

**Issues**:
- {List any type complexity concerns or "None - ✅"}

---

### Test Execution Budget

**Estimated Unit Test Runtime**: {seconds}
**Estimated Integration Test Runtime**: {seconds}
**Estimated Total Runtime**: {seconds}
**CI Timeout Limit**: {typically 120-600s}

**Within Budget**: {✅ / ⚠️}

**Long-Running Tests** (> 5s):
- {list test IDs or "None"}

---

### Commit Size Validation

**Total Commits**: {count}
**Largest Commit**: Commit {number} - {estimated LOC/files}
**Estimated Max Commit Size**: {< 500 LOC ✅ / > 500 LOC ⚠️}

**Atomicity**: {✅ PASS / ⚠️ REVIEW NEEDED}

**Issues**:
- {List any commits that may be too large or "None - ✅"}

---

## Traceability Matrix

| AC | Tests Covering | Risks Addressed | Commits Implementing | Status |
|----|----------------|-----------------|----------------------|--------|
| AC1 | {test-ids}    | {risk-ids}      | {commit-numbers}     | ✅/⚠️/❌ |
| AC2 | {test-ids}    | {risk-ids}      | {commit-numbers}     | ✅/⚠️/❌ |
| ... | ...           | ...             | ...                  | ...    |

**Legend**:
- ✅ Fully covered (tests + commits + risk mitigation)
- ⚠️ Partially covered (missing one element)
- ❌ Not covered (critical gap)

---

## Readiness Assessment

### Blockers (Must Fix Before Implementation) ❌

{List blockers or "None - ✅"}

**Impact**: {Explanation of why these block implementation}

---

### Warnings (Should Address Before Implementation) ⚠️

{List warnings or "None - ✅"}

**Impact**: {Explanation of risks if not addressed}

---

### Recommendations 💡

{List recommendations for improvement}

---

## Quality Gates Integration

**Gate Status for Story {story-number}**: {READY / NEEDS REVISION / BLOCKED}

### YAML Block for Gate File

```yaml
atomic_plan_assessment:
  date: {YYYY-MM-DD}
  artifacts_validated: {count}/4
  blockers: {count}
  warnings: {count}
  readiness: {READY / NEEDS REVISION / BLOCKED}
  ac_coverage: {count}/{total}
  test_coverage: {count}/{total}
  critical_risk_coverage: {count}/{total}
  test_pyramid_compliance: {true/false}
  recommendation: "{one-sentence summary}"
```

---

## Next Steps

### If Status = READY ✅
1. Proceed with implementation: `/atomic-commit {story-number} 1`
2. Follow commit plan sequentially
3. Run verification commands after each commit
4. Update gate file with assessment YAML block

### If Status = NEEDS REVISION ⚠️
1. Address warnings documented above
2. Update affected artifacts (story, test design, or commit plan)
3. Re-run assessment: `/review-atomic-plan {story-number}`
4. Proceed when status = READY

### If Status = BLOCKED ❌
1. **STOP** - Do not proceed with implementation
2. Address all blockers documented above
3. Revise planning artifacts as needed
4. Re-run planning phase if necessary
5. Re-run assessment after fixes

---

## Appendix: Validation Checklist Details

{Include full checklist results with ✅/❌ for each item}

### Story Artifact Checklist
- {✅/❌} Story status is "Draft" or "Ready"
- {✅/❌} Acceptance criteria present (≥3 ACs)
- ...

### Risk Profile Checklist
- {✅/❌} Executive summary present
- {✅/❌} Critical/high risks identified
- ...

### Test Design Checklist
- {✅/❌} Test strategy overview present
- {✅/❌} Test IDs follow convention
- ...

### Commit Plan Checklist
- {✅/❌} Total commit count reasonable (5-15)
- {✅/❌} Each commit has verification commands
- ...

### Implementation Readiness Checklist

**Dependency Pre-Check:**
- {✅/❌} Prerequisite stories merged
- {✅/❌} Required base classes/modules exist
- {✅/❌} Python version requirement met
- {✅/❌} External tool dependencies documented

**Test Data Feasibility:**
- {✅/❌} All fixtures are synthetic or cached
- {✅/❌} No fixtures require live API calls in unit tests
- {✅/❌} Historical data fixtures have clear provenance
- {✅/❌} Large dataset fixtures can be generated/cached

**Type System Compatibility:**
- {✅/❌} Complex nested types compatible with Python version
- {✅/❌} Forward reference strategy documented if needed
- {✅/❌} Mypy strict mode compatibility verified

**Test Execution Budget:**
- {✅/❌} Estimated runtime within CI timeout
- {✅/❌} Long-running tests (> 5s) identified
- {✅/❌} NDU-dependent tests have skip conditions

**Commit Size Validation:**
- {✅/❌} All commits estimated < 500 LOC
- {✅/❌} Largest commit identified
- {✅/❌} No commits combine unrelated features
- {✅/❌} Commit dependencies are minimal

---

**End of Assessment Report**
```

## Output Format

After writing the assessment report, provide a summary to the user:

```
## Atomic Plan Assessment Complete

**Story**: {story-number} - {story-title}
**Status**: {READY / NEEDS REVISION / BLOCKED}

### Results Summary
- ✅ Artifacts Found: {count}/4
- ⚠️ Warnings: {count}
- ❌ Blockers: {count}

### Assessment Details
**Location**: `docs/qa/assessments/{story-number}-atomic-plan-assessment-{YYYYMMDD}.md`

{If READY}
✅ **All planning artifacts are complete and consistent.**
You can proceed with implementation:
```
/atomic-commit {story-number} 1
```

{If NEEDS REVISION}
⚠️ **Planning artifacts have warnings that should be addressed.**
Review the assessment document and address warnings before proceeding.

{If BLOCKED}
❌ **Critical issues found - implementation is BLOCKED.**
Review blockers in the assessment document and fix before proceeding.
```

## Important Notes

- Be thorough but pragmatic - minor formatting issues are warnings, not blockers
- If test IDs don't follow exact convention but are consistent, that's acceptable (warning only)
- If story is simple with low risk, fewer tests is acceptable (document rationale)
- Always provide actionable feedback - explain WHY something is a blocker/warning
- Traceability gaps for P2 tests are warnings, not blockers
- Missing documentation is a warning unless it affects implementation clarity
