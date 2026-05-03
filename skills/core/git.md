# git.md

## Purpose

Define all version control standards, branching strategy, commit conventions,
PR rules, and GitHub CLI (`gh`) operations the agent must follow on every project.
This skill is loaded on every task — no exceptions.

---

## Conventions

### 1. Branching Strategy

Follow a **Feature Branch Workflow** — `main` is always stable and deployable.
No direct commits to `main` ever.

| Branch Type | Naming Convention | Description                          |
| ----------- | ----------------- | ------------------------------------ |
| Main        | `main`            | Production-ready. No direct commits. |
| Feature     | `feature/<name>`  | New features or enhancements.        |
| Bugfix      | `fix/<name>`      | Bug fixes for existing features.     |
| Hotfix      | `hotfix/<name>`   | Critical production fixes.           |
| Docs        | `docs/<name>`     | Documentation only changes.          |

Always create branches from the latest `main`:

```bash
git checkout main && git pull origin main && git checkout -b feature/my-new-feature
```

---

### 2. Commit Conventions

Use **Conventional Commits** — every commit must follow this format:

**Format:** `<type>(<scope>): <description>`

| Type       | When to Use                                     |
| ---------- | ----------------------------------------------- |
| `feat`     | A new feature                                   |
| `fix`      | A bug fix                                       |
| `docs`     | Documentation only changes                      |
| `style`    | Formatting, whitespace — no logic changes       |
| `refactor` | Code change that is neither a fix nor a feature |
| `perf`     | Performance improvement                         |
| `test`     | Adding or correcting tests                      |
| `chore`    | Build process, tooling, dependency updates      |

**Example:**

```
feat(auth): add jwt token validation
fix(orders): resolve null pointer on empty cart
```

---

### 3. Pull Request (PR) Rules

1. **Atomicity** — one PR per feature or bug fix, keep them small
2. **Description** — every PR must include a summary of changes and a linked issue
3. **Review** — at least one approval required before merging
4. **CI/CD** — all status checks must pass (lint, tests, build) before merge
5. **Cleanup** — delete the feature branch immediately after merging

---

### 4. .gitignore Standards

Every project must include a `.gitignore` from the start.

**Always exclude:**

- **Secrets:** `.env`, `*.pem`, `creds.json`, `*.key`
- **Dependencies:** `node_modules/`, `vendor/`, `venv/`, `.venv/`
- **Build artifacts:** `dist/`, `build/`, `*.exe`, `__pycache__/`, `*.pyc`
- **OS files:** `.DS_Store`, `Thumbs.db`
- **IDE config:** `.vscode/`, `.idea/`
- **Logs:** `*.log`, `logs/`

---

### 5. GitHub CLI (`gh`) Operations

All GitHub operations use the `gh` CLI.
Always prefer `--json` with `--jq` for structured, parseable output.

#### Authentication & Setup

```bash
gh auth login                                        # Interactive OAuth login
gh auth status                                       # Check current auth and active account
gh auth refresh --scopes repo,read:org,workflow      # Add missing scopes
```

#### Reference Routing

| Intent                                     | Command                                  |
| ------------------------------------------ | ---------------------------------------- |
| Create / list / view / edit / close issues | `gh issue [list/view/create/edit/close]` |
| Create / list / view / review / merge PRs  | `gh pr [list/view/create/review/merge]`  |
| List runs, view logs, watch workflows      | `gh run [list/view/watch]`               |
| Create / fork / clone / view repos         | `gh repo [create/fork/clone]`            |
| Create / list / view / edit / delete gists | `gh gist [create/list/view/edit/delete]` |

#### Calibration Rules

1. **Prefer `--json` over text output** — text formats are unstable across versions
2. **Use `--jq` to filter at source** — minimize output and reduce data transfer
3. **Prefer high-level commands** — use `gh issue create` instead of raw `gh api` where possible
4. **Always use `--repo` explicitly** — never rely on implicit detection in scripts or CI

#### Error Handling

| Error                 | Cause                 | Resolution                         |
| --------------------- | --------------------- | ---------------------------------- |
| 401 Unauthorized      | Token expired         | Run `gh auth login`                |
| 403 Forbidden         | Missing scopes        | Run `gh auth refresh --scopes ...` |
| 404 Not Found         | Repo/resource missing | Check repo name and permissions    |
| 429 Too Many Requests | Rate limited          | Wait or reduce request frequency   |

#### Known Limitations

- **Network required** — no offline mode for `gh` commands
- **Secrets are write-only** — `gh secret set` works but secrets cannot be retrieved
- **Artifact retention** — workflow artifacts expire after 90 days by default

---

## Anti-Patterns

- Never commit directly to `main` — always use a branch
- Never write vague commit messages like `"fix stuff"` or `"update"`
- Never commit `.env` files, secrets, or credentials under any circumstance
- Never merge a PR without passing CI checks
- Never leave feature branches alive after merging
- Never skip the `.gitignore` setup at project initialization
- Never use text output from `gh` in scripts — always use `--json --jq`
- Never rely on implicit repo detection in automated workflows

---

## Ready-to-Use Prompt

```
Task: [describe the git operation needed — branch, commit, PR, CI check, etc.]
Skill: git

PRE-FLIGHT CHECKLIST (run before every commit):
1. Run `git status` — audit all untracked and modified files
2. Confirm no .env, secrets, or credentials are staged
3. Confirm branch name follows naming convention
4. Review diff with `git diff --staged` before committing
5. Apply code-review.md gate before finalizing

BRANCHING:
- Always branch from latest main
- Follow naming convention: feature/<name>, fix/<name>, hotfix/<name>

COMMITTING:
- Use Conventional Commits format: <type>(<scope>): <description>
- One logical unit of work per commit
- Never bundle unrelated changes in one commit

PR:
- Include summary of changes and linked issue
- All CI checks must pass before merge
- Delete branch after merge

GITHUB CLI:
- Use gh CLI for all GitHub operations
- Always use --json --jq for structured output
- Always pass --repo explicitly in scripts

IF BLOCKED:
- Document the blocker as a comment
- Do not skip the pre-flight checklist under any circumstance
- Log your reasoning for any non-obvious decision

DONE WHEN:
- Branch follows naming convention
- All commits follow Conventional Commits
- Pre-flight checklist passed
- PR created with description and linked issue
- CI checks passing
```
