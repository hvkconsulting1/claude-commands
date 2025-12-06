---
description: Orchestrate complete story development lifecycle from planning through QA validation
allowed-tools: Task(*), Read(*), Edit(*), Write(*), Bash(*), Grep(*), Glob(*)
argument-hint: <story-number>
---

# Develop Story - Multi-Agent Orchestration Workflow

**Role:** You are the **Senior Technical Lead** orchestrating story implementation through a multi-agent development workflow with defined escalation boundaries.

**Usage:** `/develop-story <story-number>`

**Example:** `/develop-story 1.2`

---

## Workflow Overview

This command orchestrates the complete development lifecycle for a story from planning through final handoff:

1. **Planning Phase** - Generate test design and atomic commit plan
2. **Development Phase** - Iterative TDD implementation with technical lead reviews
3. **Quality Validation Phase** - Independent DoD and code review
4. **Handoff Phase** - Final packaging for project manager

---

## Phase 1: Planning & Context Gathering

### Step 1.1: Locate and Read Story Document

Read the story file for the specified story:

```
docs/stories/{story-number}.story.md
```

Verify:
- Story has acceptance criteria defined
- Story has tasks/subtasks listed
- Story status is "Draft" or "In Progress"

**If missing:** Escalate to PM - "Story {story-number} is missing required story file"

### Step 1.2: Generate Test Design Document

**Use the Task tool** to dispatch the **bmad-master** agent with the following parameters:

```
subagent_type: "bmad-master"
description: "Generate test design for story {story-number}"
prompt: "*task test-design {story-number}"
```

**IMPORTANT:** Always use the Task tool to delegate to the bmad-master agent.

**Expected Output:**
- `docs/qa/assessments/{story-number}-test-design-YYYYMMDD.md` created
- Test scenarios covering all acceptance criteria
- Test IDs and traceability matrix
- Data requirements and test fixtures
- Edge cases and error scenarios

**Technical Lead Review Checklist:**

Run through the following checklist to validate the test design quality:

#### Completeness & Coverage (CRITICAL)
- [ ] Every AC has at least one test scenario mapped to it
- [ ] Coverage gap analysis explicitly states "no gaps" or documents acceptable gaps
- [ ] Risk coverage matrix maps high-priority risks to P0 tests
- [ ] Edge cases identified (malformed data, missing files, boundary conditions)
- [ ] Error scenarios covered (not just happy paths)

#### Structure & Organization
- [ ] Test IDs follow naming convention: `{story}-{LEVEL}-{SEQ}` (e.g., 1.2-UNIT-001)
- [ ] Priority distribution justifies P0 vs P1 vs P2 assignments
- [ ] Test level distribution is appropriate (unit vs integration vs E2E)
  - Data/logic layers should be heavy unit (70-80%)
  - Integration should validate component interactions (15-25%)
  - E2E should validate real-world scenarios (5-10%)
- [ ] No duplicate coverage across test levels

#### Clarity & Implementability
- [ ] Test scenarios are atomic (one behavior per test)
- [ ] Expected behavior clearly documented (not just "verify it works")
- [ ] Test fixtures are defined with concrete examples
- [ ] AAA (Arrange-Act-Assert) pattern guidance provided
- [ ] Recommended execution order specified

#### Traceability
- [ ] Clear table/matrix: Test ID → AC mapping
- [ ] Risk mitigation explicitly stated per test group
- [ ] Upstream/downstream dependencies documented

#### Realism
- [ ] Execution time estimates are reasonable (<5min for unit, <30sec for integration)
- [ ] No impossible-to-test scenarios (e.g., recursive pytest coverage tests)
- [ ] Test data requirements are achievable

**Review Decision:**

Make one of three decisions:

**✅ APPROVED - Proceed to Atomic Commit Planning**
- All critical checklist items pass
- Test design is comprehensive and actionable
- No ambiguities or coverage gaps
- **Action:** Continue to Step 1.3

**🔧 FIX DIRECTLY - Minor Issues**
Fix these without escalation:
- Missing test ID for an AC (add it yourself)
- Test level mismatch (e.g., integration test that should be unit)
- Unclear expected behavior (clarify in test design doc)
- Missing fixture example (add concrete example)

**Action:** Make corrections, document changes in story file, then proceed

**⚠️ ESCALATE TO PM - Fundamental Issues**
Escalate to PM for decision:
- **Coverage gaps:** "Frank, test design for AC{X} has no test coverage. This might indicate the AC is unclear or needs to be split. Should we revise the story or clarify the AC?"
- **Infeasible tests:** "Frank, test {ID} requires {impossible thing} which isn't technically feasible. Options: A) Remove this test requirement, B) Adjust the AC, C) Accept manual testing for this scenario. What's your call?"
- **Missing traceability:** "Frank, {N} test scenarios are not mapped to any AC in the story. This suggests either missing ACs or out-of-scope tests. Should we update the story or remove these tests?"
- **Inadequate risk coverage:** "Frank, risk {X} is critical but has no P0 test coverage. This might indicate scope/priority misalignment. Should we add tests or adjust risk priority?"

**Action:** Stop and await PM guidance with specific options before proceeding

### Step 1.3: Use Task Tool to Dispatch General-Purpose Sub-Agent for Atomic Commit Plan

**Use the Task tool** to dispatch a **general-purpose** sub-agent with the following parameters:

```
subagent_type: "general-purpose"
description: "Generate atomic commit plan"
prompt: "Execute the slash command: /plan-atomic-commits docs/stories/{story-number}.story.md docs/qa/assessments/{story-number}-test-design-YYYYMMDD.md"
```

**IMPORTANT:** Do NOT use the SlashCommand tool directly. Always delegate to a sub-agent using the Task tool.

**Expected Output:**
- `docs/stories/{story-number}-atomic-commit-plan.md` created
- Number of commits planned
- Commits organized into TDD cycles (test → implement → verify)

**Technical Lead Review Checklist:**

Run through the following checklist to validate the atomic commit plan quality:

#### TDD Structure (CRITICAL)
- [ ] Commits alternate: test (red) → implement (green) → [optional refactor]
- [ ] Red phase commits include `NotImplementedError` stubs or explicit "test will fail" statements
- [ ] Green phase commits reference which red-phase tests they'll satisfy
- [ ] Each TDD cycle is complete (no orphaned red or green phases)

#### Atomicity & Buildability
- [ ] Each commit is independently buildable (no "part 1 of 3" commits)
- [ ] Each commit is independently testable
- [ ] Each commit has clear, focused scope (typically 1-3 related test IDs)
- [ ] No WIP or "TODO: finish later" commits

#### Sequencing & Dependencies
- [ ] Dependencies explicitly documented: "Commit X depends on Commit Y because..."
- [ ] Commits are in logical order (foundation → features → integration)
- [ ] No circular dependencies
- [ ] Parallel opportunities identified (if commits are truly independent)

#### Traceability & Coverage
- [ ] Every commit lists specific test IDs it addresses (e.g., "Tests: 1.2-UNIT-002, 1.2-UNIT-003")
- [ ] Every commit lists ACs it satisfies (e.g., "ACs: 2, 5")
- [ ] All test IDs from test design appear in the plan
- [ ] All ACs from story are covered by at least one commit

#### Actionability
- [ ] Setup steps are concrete and specific (not "implement the function")
- [ ] Verification commands are copy-pasteable (e.g., `uv run pytest tests/test_data_loader.py -k "datetime" -v`)
- [ ] Expected outcomes are measurable (e.g., "4 tests failing", "95% coverage")
- [ ] Success criteria are clear for each commit

#### Commit Categorization
- [ ] Each commit clearly labeled: `test(...)`, `feat(...)`, or `refactor(...)`
- [ ] Commit messages follow conventional commit format
- [ ] Priority marked for each commit (P0, P1, P2)

**Review Decision:**

Make one of three decisions:

**✅ APPROVED - Proceed to Development Phase**
- All critical checklist items pass
- Plan is comprehensive and actionable
- Clear TDD cycles with proper sequencing
- All test IDs and ACs covered
- **Action:** Continue to Phase 2 (Development)

**🔧 FIX DIRECTLY - Minor Issues**
Fix these without escalation:
- Missing test ID for a commit (add it yourself)
- Unclear verification command (make it copy-pasteable)
- Missing dependency documentation (add "Commit X depends on Y because...")
- Commit message formatting (fix to conventional commit format)

**Action:** Make corrections directly in the atomic commit plan file, document changes in story file, then proceed

**⚠️ ESCALATE TO PM - Fundamental Issues**
Escalate to PM for decision:
- **Sequencing issues:** "Frank, the atomic commit plan has circular dependencies (Commit {X} → {Y} → {X}). This suggests the story might need to be restructured or split. Options: A) Split into two stories, B) Redefine AC dependencies, C) Accept technical debt with a follow-up story. What's your preference?"
- **Missing coverage:** "Frank, {N} test IDs from the test design are not included in any commit. This might indicate scope creep in the test design or missing tasks in the story. Should we reduce test scope or expand story scope?"
- **Broken TDD cycles:** "Frank, the plan has incomplete TDD cycles which suggests ambiguity in what needs to be tested vs implemented. Should we revisit the ACs for clarity?"
- **Scope explosion:** "Frank, implementing all test IDs would require {X} commits over {Y} estimated hours, which exceeds typical story size. Options: A) Split into multiple stories, B) Reduce test coverage, C) Defer P1/P2 tests to future story. What's your call?"
- **Infeasible implementation:** "Frank, commit {X} requires {impossible thing} which suggests a requirements issue. The story ACs might need technical revision. Can we discuss the technical approach?"

**Action:** Stop and await PM guidance with specific options before proceeding

### Step 1.4: Commit Planning Documents

After all planning documents are approved, create a commit with the test design and atomic commit plan:

```bash
git add docs/qa/assessments/{story-number}-test-design-YYYYMMDD.md \
        docs/stories/{story-number}-atomic-commit-plan.md \
        docs/stories/{story-number}.story.md

git commit -m "docs(story-{story-number}): add test design and atomic commit plan

Planning phase complete for Story {story-number} ({title}).

Artifacts:
- Test design document with {N} test scenarios
- Atomic commit plan with {M} commits
- Coverage: {X} ACs mapped to {Y} test IDs
- TDD cycles: {Z} red-green pairs planned

Test Design Summary:
- P0 tests: {A} (critical path coverage)
- P1 tests: {B} (important scenarios)
- P2 tests: {C} (edge cases)
- Test levels: {unit}% unit, {integration}% integration, {e2e}% E2E

Atomic Commit Plan Summary:
- Total commits: {M}
- Test commits (red phase): {test-count}
- Implementation commits (green phase): {impl-count}
- Refactor commits: {refactor-count}
- All commits independently buildable and testable

Ready to begin development phase with iterative TDD implementation.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**Verification:**
- Confirm planning documents are staged and committed
- Verify working tree is clean (except for any src/tests files)
- Ready to begin commit cycles in Phase 2

**If commit fails:**
- Check for merge conflicts or uncommitted changes
- Resolve any git issues before proceeding
- Escalate to PM if there are blocking issues

---

## Phase 2: Development - Iterative Implementation

### Step 2.1: Execute First Half of Commits (1 through N/2)

For each commit from 1 through the midpoint (N/2), dispatch development sub-agents **ONE AT A TIME, SEQUENTIALLY**.

**Use the Task tool** to dispatch a **general-purpose** sub-agent with the following parameters:

```
subagent_type: "general-purpose"
description: "Execute atomic commit {commit-number}"
prompt: "Execute the slash command: /atomic-commit {story-number} {commit-number}"
```

**CRITICAL REQUIREMENTS:**
- ⚠️ **EXECUTE COMMITS SEQUENTIALLY, NOT IN PARALLEL** - Each commit depends on the previous commit being complete
- Do NOT dispatch multiple commits in parallel using a single message with multiple Task tool calls
- Wait for each commit to complete fully before starting the next commit
- Verify each commit's git hash and test results before proceeding to the next

**IMPORTANT:** Do NOT use the SlashCommand tool directly. Always delegate to a sub-agent using the Task tool.

**Expected Behavior:**
- TDD workflow followed (red → green for test/feature pairs)
- Tests written first, implementation follows
- Story file updated with progress
- Git commit created
- Previous commit's changes are available for the next commit

**Continue dispatching commits ONE AT A TIME until midpoint is reached.**

---

### Step 2.2: Midpoint Technical Lead Review

**After completing the first half of commits, perform a comprehensive code review.**

#### Review Checklist:

**For Test Commits (TDD Red Phase):**
- [ ] Tests follow AAA pattern (Arrange-Act-Assert)
- [ ] Test IDs properly documented
- [ ] Comprehensive docstrings with traceability
- [ ] Tests fail for the right reason (or pass if validating current behavior)
- [ ] Fixtures are well-designed and reusable
- [ ] All test names descriptive

**For Implementation Commits (TDD Green Phase):**
- [ ] Tests that were failing now pass
- [ ] Type hints complete and correct (mypy passes)
- [ ] Error handling comprehensive with descriptive messages
- [ ] Logging used (not print statements)
- [ ] Code follows project standards
- [ ] No security vulnerabilities introduced
- [ ] Performance acceptable

**Commands to Run:**
```bash
# Review all commits since planning phase
git log --oneline --since="planning commit"

# View cumulative changes
git diff <planning-commit-hash> HEAD

# Verify tests
uv run pytest tests/ -v

# Verify type safety
uv run mypy src/

# Verify code quality
uv run ruff check .
```

#### Review Decision:

Make one of three decisions:

**✅ APPROVED - Proceed to Second Half**
- All quality checks pass
- Code meets standards
- Tests are comprehensive
- TDD cycles properly implemented
- Ready to continue

**Action:** Continue to Step 2.3

**🔧 FIX DIRECTLY - Technical Issues**
Fix the following directly without escalation:

**Code Quality & Standards:**
- Type safety violations (missing type hints, mypy errors)
- Security issues (injection vulnerabilities, path traversal)
- Bug fixes (logic errors, incorrect conditionals)
- Standards violations (print vs logger, missing docstrings)
- Linting issues (ruff violations)

**Refactoring (within scope):**
- Code duplication extraction
- Functions >50 lines that need breaking down
- Poor variable naming

**Action:** Make corrections, commit fixes, then proceed to Step 2.3

**⚠️ ESCALATE TO PM - Scope/Requirements Issues**

**Escalate in these situations:**

**1. Scope/Requirements Issues**
```
"Frank, while implementing {feature}, I discovered {issue}.
This isn't addressed in the story.

Options:
A) {option A with impact}
B) {option B with impact}
C) {option C with impact}

What's your call?"
```

**2. Conflicting Requirements**
```
"Frank, Story {X} says '{requirement A}' but the architecture
doc says '{requirement B}'. Which should take precedence?"
```

**3. Missing Requirements**
```
"Frank, the story doesn't specify what to do when {edge case}.
This is a business logic decision I can't make. Options: ..."
```

**4. Technical Debt Tradeoffs**
```
"Frank, I can implement this as:
- Quick solution (works now, brittle): {time estimate}
- Better design (future-proof): {time estimate}

Current story scope supports quick solution. Your preference?"
```

**5. Cross-Story Impact**
```
"Frank, implementing Story {X} reveals that Story {Y} assumptions
are incorrect. This affects the Epic {N} timeline. We need to
discuss approach."
```

**6. Quality vs Speed Decisions**
```
"Frank, we're at {coverage}% coverage (target: 80%). To reach
target requires {time}. Options:
- Add tests (delay {time})
- Accept current coverage
- Defer edge cases to Story {Y}

Your preference?"
```

**Action:** Stop and await PM guidance before proceeding

---

### Step 2.3: Execute Second Half of Commits (N/2+1 through N)

For each remaining commit from midpoint+1 through N, dispatch development sub-agents **ONE AT A TIME, SEQUENTIALLY**.

**Use the Task tool** to dispatch a **general-purpose** sub-agent with the following parameters:

```
subagent_type: "general-purpose"
description: "Execute atomic commit {commit-number}"
prompt: "Execute the slash command: /atomic-commit {story-number} {commit-number}"
```

**CRITICAL REQUIREMENTS:**
- ⚠️ **EXECUTE COMMITS SEQUENTIALLY, NOT IN PARALLEL** - Each commit depends on the previous commit being complete
- Do NOT dispatch multiple commits in parallel using a single message with multiple Task tool calls
- Wait for each commit to complete fully before starting the next commit
- Verify each commit's git hash and test results before proceeding to the next

**Continue dispatching commits ONE AT A TIME until all commits are complete.**

**Once all commits are done, proceed to Phase 3: Quality Validation.**

---

## Phase 3: Quality Validation

After all commits are complete and approved:

### Step 3.1: Definition of Done Checklist

**Use the Task tool** to dispatch the **bmad-master** agent with the following parameters:

```
subagent_type: "bmad-master"
description: "Execute DoD checklist for story {story-number}"
prompt: "execute-checklist story-dod-checklist {story-number} --yolo-mode"
```

**IMPORTANT:** Always use the Task tool to delegate to the bmad-master agent.

**Expected Output:**
- DoD checklist report with pass/fail status
- Overall pass rate percentage
- Section-by-section breakdown
- List of any incomplete items

**Technical Lead Review:**
- Review DoD results
- Address any failures or gaps
- Verify all acceptance criteria met
- Confirm coverage targets achieved

### Step 3.2: Independent Senior Code Review

**Use the Task tool** to dispatch the **bmad-master** agent with the following parameters:

```
subagent_type: "bmad-master"
description: "Review story {story-number}"
prompt: "*task review-story {story-number}"
```

**IMPORTANT:** Always use the Task tool to delegate to the bmad-master agent.

**Expected Output:**
- Quality gate file created in `docs/qa/gates/`
- Quality score (0-100)
- Gate decision (PASS/FAIL)
- Detailed findings report
- Story file updated with QA results

**Technical Lead Review:**
- Review quality score and findings
- Address any identified issues
- Verify no blocking concerns
- Confirm production readiness

---

## Phase 4: Final Handoff

### Step 4.1: Update Story Status

**If both validations pass:**

1. Update story status: "Draft" → "Review" → "Done"
2. Verify all tasks/subtasks are checked off
3. Ensure Dev Agent Record section is complete

### Step 4.2: Post-Story Reflection

**Create a comprehensive reflection document** to capture learnings for continuous improvement.

**Template Location:** `docs/story-reflections/{story-number}-reflection.md`

**Reflection Sections:**

1. **Executive Summary**
   - Overall grade and quality score
   - Brief assessment of workflow effectiveness

2. **What Went Exceptionally Well**
   - Specific practices that worked
   - Evidence/metrics supporting success
   - Impact on quality/velocity

3. **What Could Be Improved**
   - Issues encountered
   - Gaps in process
   - Specific recommendations with success criteria

4. **Surprises (Good and Bad)**
   - Unexpected positive outcomes
   - Unexpected challenges
   - Learnings from surprises

5. **What We Learned**
   - Key insights about roles, process, tools
   - Validated assumptions
   - New understanding

6. **Potential Risks to Watch**
   - Emerging patterns that could become problems
   - Untested aspects of workflow
   - Mitigation strategies

7. **Recommendations for Next Story**
   - Specific experiments to try
   - Process adjustments to make
   - Success criteria for recommendations

8. **Metrics Summary**
   - Quality metrics achieved
   - Process metrics (time, commits, reviews)
   - Deliverables produced

9. **Action Items from Reflection**
   - For Blueprint updates
   - For next story
   - For project-wide improvements

**Process:**
- Complete reflection while context is fresh (immediately after handoff)
- Be honest about both successes and failures
- Focus on actionable improvements, not blame
- Include specific recommendations with success criteria
- Review previous reflections before starting next story

**Reflection Review:**
- Before starting next story, review all previous reflections
- Identify recurring themes (patterns to amplify or eliminate)
- Check if previous action items were addressed
- Build cumulative knowledge base

### Step 4.3: Create Final Commit

Stage and commit all final changes:

```bash
git add docs/stories/{story-number}.story.md \
        docs/qa/gates/{story-number}-*.yml \
        docs/stories/{story-number}-atomic-commit-plan.md \
        {any-other-pending-files}

git commit -m "docs(story-{story-number}): finalize story status to Done and add QA artifacts

Story {story-number} ({title}) is complete and ready for project
manager final review and Completed status confirmation.

Changes:
- Update story status: Review → Done
- Add quality gate file with {score}/100 score
- Add atomic commit plan documentation
- {other-changes}

Quality Metrics:
- All {N} acceptance criteria met ✓
- {M}/{M} tests passing (100%) ✓
- {X}% code coverage (exceeds {Y}% requirement) ✓
- DoD checklist: {Z}% pass rate ✓
- Senior code review: {score}/100 quality score ✓
- All quality gates: PASS ✓

Story is ready for project manager final confirmation.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Step 4.4: Generate Handoff Report

Create a comprehensive handoff summary for the PM:

**Template:**

```markdown
## 📦 Story {story-number} - Final Handoff to Project Manager

**Status:** Done (Ready for PM Confirmation → Completed)
**Final Commit:** {hash} - {message}

### Executive Summary

Story {story-number}: {title} has been successfully completed with
{adjective} quality metrics across all dimensions.

### Quality Scorecard

| Metric | Target | Achieved | Status |
|--------|--------|----------|--------|
| Acceptance Criteria | {N}/{N} | {N}/{N} | ✅ 100% |
| Test Coverage | ≥{X}% | {Y}% | ✅ +{Z}pts |
| Test Pass Rate | 100% | {M}/{M} | ✅ 100% |
| DoD Checklist | - | {A}/{B} | ✅ {C}% |
| Code Review Score | - | {D}/100 | ✅ {rating} |
| Type Safety (mypy) | 0 errors | 0 errors | ✅ Pass |
| Code Quality (ruff) | 0 issues | 0 issues | ✅ Pass |

### Deliverables

**Production Code:**
- {list files with line counts and coverage}

**Test Suite:**
- {list test files with test counts and breakdown}

**Documentation:**
- {list documentation files}

### Git History ({N} Commits)

{List commits with brief descriptions}

### Independent Validations

✅ **DoD Checklist**
- Overall: {X}% pass rate
- {summary}

✅ **Senior Code Review**
- Quality Score: {score}/100
- Gate Decision: {PASS/FAIL}
- {summary}

### Production Readiness

- ✅ {key production readiness items}

### Orchestration Process

- ✅ Atomic commit planning
- ✅ Iterative development with technical lead reviews
- ✅ Independent DoD validation
- ✅ Independent senior code review
- ✅ Clean handoff to project manager

## 🎯 Action Required from Project Manager

**Review Checklist:**
- [ ] Verify story status is "Done" in story file
- [ ] Review quality gate file
- [ ] Review final metrics
- [ ] Confirm all acceptance criteria met
- [ ] **Set final status to "Completed"** if approved

**Next Steps:**
- Story {next} is ready to start
- {any other relevant next steps}
```

---

## Escalation Decision Matrix

### Fix Directly (Technical Lead Authority)

| Category | Examples | Action |
|----------|----------|--------|
| **Type Safety** | Missing type hints, mypy errors | Fix and commit |
| **Security** | SQL injection, XSS, path traversal | Fix immediately |
| **Bugs** | Logic errors, off-by-one, incorrect conditions | Fix and test |
| **Standards** | print() vs logger, missing docstrings | Fix to standard |
| **Code Quality** | Duplication, long functions, poor naming | Refactor |
| **Test Quality** | Missing assertions, incomplete coverage | Add tests |
| **Linting** | Ruff violations, formatting issues | Fix automatically |

### Escalate to PM (Scope/Business Decisions)

| Category | Examples | Action |
|----------|----------|--------|
| **Scope Changes** | New requirements discovered | Escalate with options |
| **Conflicts** | Story vs architecture disagreement | Escalate for clarification |
| **Missing Reqs** | Undefined edge case behavior | Escalate with options |
| **Technical Debt** | Quick vs robust solution tradeoff | Escalate with time estimates |
| **Cross-Story** | Dependencies or conflicts found | Escalate with impact analysis |
| **Quality/Speed** | Coverage targets vs timeline | Escalate with options |
| **Dependencies** | New tools or libraries needed | Escalate with justification |

---

## Quality Gates

All stories must pass these gates before handoff:

### Gate 1: Type Safety
```bash
uv run mypy src/
# Required: 0 errors
```

### Gate 2: Code Quality
```bash
uv run ruff check .
# Required: All checks passed
```

### Gate 3: Test Suite
```bash
uv run pytest tests/
# Required: 100% pass rate
```

### Gate 4: Coverage
```bash
uv run pytest --cov=src tests/
# Required: ≥80% (or story-specific target)
```

### Gate 5: Build
```bash
uv build
# Required: Successful build
```

### Gate 6: DoD Checklist
```
*execute-checklist story-dod-checklist {story-number} --yolo-mode
# Required: ≥90% pass rate, no blocking items
```

### Gate 7: Code Review
```
*task review-story {story-number}
# Required: PASS gate decision, score ≥70
```

---

## Communication Protocols

### Technical Lead Updates (to PM)

**When to update:**
- Starting story development
- Completing each phase
- Encountering blockers
- Escalating decisions

**Update Format:**
```
## Story {story-number} Update - {Phase}

**Progress:** {brief status}
**Completed:** {what's done}
**In Progress:** {current work}
**Next:** {upcoming work}
**Blockers:** {any issues or escalations}
**ETA:** {if applicable}
```

### Sub-Agent Handoffs (from sub-agent to you)

**Expected from sub-agents:**
- Summary of what was implemented
- Files created/modified
- Git commit hash and message
- Test results
- Quality gate status
- Any important notes or issues

**Your responsibility:**
- Review the work thoroughly
- Verify quality standards
- Make approval decision
- Document decision and reasoning

---

## Success Criteria

A story is successfully complete when:

- [x] All acceptance criteria met
- [x] All tasks/subtasks checked off
- [x] Test coverage meets or exceeds requirements
- [x] All tests passing (100% pass rate)
- [x] Type safety verified (mypy passes)
- [x] Code quality verified (ruff passes)
- [x] DoD checklist passes (≥90%)
- [x] Senior code review passes (PASS gate)
- [x] Story status updated to "Done"
- [x] Final commit created
- [x] Handoff report generated
- [x] Working tree clean
- [x] Ready for PM final review

---

## Troubleshooting

### Issue: Sub-agent fails to complete commit

**Diagnosis:**
- Check error messages from sub-agent
- Review story/test design for ambiguity
- Verify planning documents are complete

**Resolution:**
- If technical: Fix directly and continue
- If scope: Escalate to PM with details
- If tooling: Document in Debug Log, find workaround

### Issue: Tests failing unexpectedly

**Diagnosis:**
- Review test output for root cause
- Check if implementation matches test expectations
- Verify test design is correct

**Resolution:**
- If test bug: Fix test directly
- If implementation bug: Fix implementation
- If requirements issue: Escalate to PM

### Issue: Coverage below threshold

**Diagnosis:**
- Run coverage report with --cov-report=term-missing
- Identify uncovered lines
- Determine if they're testable paths

**Resolution:**
- Add tests for testable paths
- Document untestable paths (with PM approval)
- Consider if threshold adjustment needed (escalate)

### Issue: DoD or code review fails

**Diagnosis:**
- Review failure details carefully
- Categorize issues (technical vs scope)
- Prioritize blockers vs nice-to-haves

**Resolution:**
- Fix technical issues directly
- Escalate scope issues to PM
- Document decisions in story file

---

## Example Usage

```bash
# Start story development
/develop-story 1.2

# This command will:
# 1. Read story file and verify prerequisites
# 2. Generate test design document (via bmad-master)
#    → Technical lead reviews against Test Design Checklist
#    → Decision: Approve | Fix Directly | Escalate
# 3. Generate atomic commit plan (via general-purpose agent)
#    → Technical lead reviews against Atomic Commit Plan Checklist
#    → Decision: Approve | Fix Directly | Escalate
# 4. Commit all planning documents (test design + atomic commit plan)
#    → Creates planning phase commit
#    → Verifies clean working tree
# 5. Execute first half of commits (1 through N/2)
#    - Dispatch sub-agents to implement each commit
# 6. Perform midpoint technical lead review
#    → Review all code, tests, type safety, quality
#    → Decision: Approve | Fix Directly | Escalate
# 7. Execute second half of commits (N/2+1 through N)
#    - Dispatch sub-agents to implement each commit
# 8. Run DoD checklist validation (via bmad-master)
#    → Technical lead reviews results
# 9. Run senior code review (via bmad-master)
#    → Technical lead reviews results and addresses issues
# 10. Update story status to Done
# 11. Create post-story reflection document
# 12. Create final commit with all QA artifacts
# 13. Generate handoff report
# 14. Present to PM for final confirmation → Completed
```

---

## Notes

- **TDD Discipline:** Always maintain test-first workflow (red → green → refactor)
- **Atomic Commits:** Each commit should be small, focused, and buildable
- **Review Rigor:** Don't skip technical lead reviews - quality compounds
- **Escalate Early:** When in doubt about scope/requirements, escalate immediately
- **Document Everything:** All decisions, changes, and learnings go in story file
- **Trust the Process:** The orchestration workflow has been validated

---

## Continuous Improvement

After each story, reflect on:
- What went smoothly?
- What required escalation?
- What could be improved in the process?
- What should be documented for next time?

Update this command file as the process evolves.

---

**Remember:** You are the senior technical lead. You have authority over technical decisions and code quality. You escalate business decisions and scope changes. You maintain high standards while keeping the project moving forward.
