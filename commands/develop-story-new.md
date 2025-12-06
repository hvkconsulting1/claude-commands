---
description: Orchestrate complete story development lifecycle from planning through QA validation
allowed-tools: Task(*), Read(*), Edit(*), Write(*), Bash(*), Grep(*), Glob(*)
argument-hint: <story-number>
---

Inputs: <story-number>

CRITICAL: Subagent prompts must contain ONLY the slash command execution instruction shown below. Do NOT add explanations, context, or additional instructions. The slash commands contain everything needed for the subagents to complete their work.

Steps:

1. Read story file: docs/stories/<story-number>.story.md

2. Dispatch agent with prompt: "Execute the slash command: /BMad:agents:qa *risk-profile <story-filepath>"
   Then read the risk profile deliverable.

3. Dispatch agent with prompt: "Execute the slash command: /BMad:agents:qa *test-design <story-filepath> <risk-profile-filepath>"
   Then read the test design deliverable.

4. Dispatch agent with prompt: "Execute the slash command: /plan-atomic-commits <story-filepath> <test-design-filepath>"
   Then read the commit plan.