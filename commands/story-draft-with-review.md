---
description: Execute comprehensive story draft review with parallel agents
allowed-tools: Task(*), Read(*), Bash(*)
argument-hint: <story_number>
---

# Story Draft Review Orchestration

Execute a comprehensive story draft review using parallel agents to identify issues and provide remediations.

**Usage:** `/story-draft-review <story_number>`

---

## Instructions for Claude

When this command is invoked with a story number, orchestrate the following review process:

### Phase 1: Story File Creation

Launch **1 general-purpose task agent** to execute the SM slash command.

**Prompt to the agent (no additional filler):**
```
execute the slash command: /BMad:agents:sm *draft <story_number> --skip-checklist
```

- Agent creates the story file
- Wait for agent to complete before proceeding to Phase 2

### Phase 2: Story Checklist Review (Parallel Execution)

After Phase 1 completes, launch **5 general-purpose task agents simultaneously** (in a single message with 5 Task tool calls).

**Prompt to each agent (no additional filler):**
```
execute the slash command: /BMad:agents:sm *execute-checklist story-draft-checklist <story_number> --yolo-mode
```

- Each agent independently reviews the story against the checklist
- Wait for ALL agents to complete before proceeding
- Collect all reports from Phase 2

### Phase 3: Story Draft Validation (Parallel Execution)

After Phase 2 completes, launch **5 general-purpose task agents simultaneously** (in a single message with 5 Task tool calls).

**Prompt to each agent (no additional filler):**
```
execute the slash command: /BMad:agents:po *validate-story-draft <story_number>
```

- Each agent independently validates the story draft
- Wait for ALL agents to complete before proceeding
- Collect all reports from Phase 3

### Results Aggregation and Interactive Remediation

After all three phases complete:

1. **Read Story File**: Read `docs/stories/<story_number>.story.md` for context
2. **Extract Issues Only**: Review all 10 agent reports (5 from Phase 2 + 5 from Phase 3) and extract ONLY issues, problems, concerns, or missing requirements
3. **De-duplicate**: Combine similar findings across agents
4. **Organize**: Group related issues together and prioritize by severity

**If no issues found**: Report "No issues found. Story draft is ready." and exit.

**If issues found**, process each issue interactively:

For each issue in the list:

1. **Present Issue**: Show the issue number (e.g., "Issue 1 of 5") and clear description of the problem
2. **Analyze Remediation Options**: Determine 2-4 specific remediation approaches for this issue
3. **Provide Recommendation**: Recommend one approach with reasoning based on:
   - Alignment with project architecture (ADR references)
   - Impact on story acceptance criteria
   - Risk and effort trade-offs
   - Consistency with existing patterns in the codebase
4. **Present Options**: Use AskUserQuestion to let the user choose:
   - Each remediation option as a choice (with description of what it entails)
   - "Skip this issue" option
5. **Execute Remediation**: If user selects a remediation option (not skip):
   - Read the story file
   - Apply the selected remediation by editing the story file
   - Confirm completion: "Applied remediation: [brief description]"
6. **Move to Next Issue**: Continue to the next issue in the list

**After all issues processed**:
- Summary: "Story Draft Review Complete for Story <story_number>"
- Report: "Processed X issues: Y remediated, Z skipped"

**IMPORTANT**:
- Only extract issues/problems from agent reports
- Do not include confirmations of what's working correctly
- Each remediation option should be specific and actionable
- Provide clear reasoning for recommendations
- Apply remediations directly to the story file when user selects them
- Process issues one at a time - don't batch them
