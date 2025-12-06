---
description: Stage and commit changes using conventional commit format
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)
argument-hint: (no arguments)
---

# Conventional Commit

Stage and commit pending changes using conventional commit message format.

## Instructions

1. Run `git status` and `git diff` in parallel to see all changes
2. Analyze the changes and determine:
   - Type: feat, fix, docs, style, refactor, test, chore, perf, ci, build, revert
   - Scope (optional): affected component or module
   - Subject: short description in imperative mood
   - Body (if needed): longer explanation of what and why
   - Breaking changes (if any): note in footer with "BREAKING CHANGE:"
3. Create a conventional commit message following this format:
   ```
   <type>(<scope>): <subject>

   <body>

   <footer>
   ```
4. Stage all relevant changes using `git add`
5. Commit with the conventional message using a heredoc:
   ```bash
   git commit -m "$(cat <<'EOF'
   <conventional commit message here>
   EOF
   )"
   ```
6. Run `git status` after committing to verify success

## Conventional Commit Guidelines

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that don't affect code meaning (formatting, etc.)
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Performance improvements
- **test**: Adding or correcting tests
- **chore**: Changes to build process or auxiliary tools
- **ci**: Changes to CI configuration files and scripts
- **build**: Changes that affect the build system or dependencies
- **revert**: Reverts a previous commit

Subject should be lowercase, no period at the end, max 50 characters.
Body should wrap at 72 characters and explain what and why vs. how.
