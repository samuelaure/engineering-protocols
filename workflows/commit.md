---
description: Commit Protocol & Code Quality Standards
---

## 1. PRE-COMMIT CHECKLIST (Non-Negotiable)

Before creating any commit, you must ensure the health of the codebase. You are responsible for formatting, testing, and typechecking YOUR code before atomic-committing it. Do not rely on CI to catch basic errors.

### Required Checks
Run the following checks locally:

1.  **Format**: Ensure all code is formatted according to project standards (Prettier).
2.  **Lint**: Run static analysis to catch potential bugs and style violations (ESLint).
3.  **Type Check**: Validate type safety (TypeScript).
4.  **Test**: Run relevant unit tests for the changes made.

### Recommended Command Sequence
Run this sequence before staging files:
```bash
npm run format && npm run lint && npm run type-check && npm run test
```

**Note on Automation**: If the project has automated hooks (like Husky or lint-staged) that perform these checks on commit, you may skip manual execution IF AND ONLY IF you are confident the hooks are active and covering the required checks. However, manual verification is always safer.

If any step fails, **fix it immediately**. Do not bypass these checks.

---

## 2. COMMIT MESSAGE CONVENTIONS

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Structure
```
<type>(<scope>): <subject>

<body (optional)>

<footer (optional)>
```

### Types
- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **chore**: Changes to the build process or auxiliary tools and libraries such as documentation generation

### Rules
- **Imperative mood**: "Add feature" not "Added feature"
- **No periods**: No dot at the end of the subject line
- **Lowercase**: Keep the subject lowercase

### Examples
- `feat(auth): implement jwt token generation`
- `fix(api): handle timeout error in user service`
- `style(ui): update button padding`

---

## 3. GIT WORKFLOW

### Atomic Commits
- Each commit should do **one thing**.
- If you have multiple changes (e.g., a bug fix and a new feature), split them into separate commits.
- This makes reverting changes and code review significantly easier.

---

## 4. AGENT EXECUTION STEPS (The "Straight Shot")

To avoid redundant commands and ensure high-fidelity commits, follow this sequence:

1.  **Discover**: Run `git status --porcelain` once to get a parseable list of changes.
2.  **Inspect**: Run `git diff` on modified files and `view_file` on new files to internalize the logic of the changes.
3.  **Group**: Determine if changes represent one atomic unit or multiple.
4.  **Verify**: Run the "Required Checks" (Linting, Testing, etc.).
5.  **Stage**: Use `git add <specific files>` instead of `git add .` to maintain atomicity.
6.  **Verify Staging**: Run `git status --porcelain` and `git diff --cached` once to confirm exactly what is going into the commit.
7.  **Commit**: Execute `git commit -m "<message>"` following Conventional Commits.

---

**Code is communication. Commits are history. Make them clean.**
