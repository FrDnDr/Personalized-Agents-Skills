

| name | git |
| :--- | :--- |
| **description** | GitHub CLI operations via `gh` for issues, PRs, Actions, releases, and REST/GraphQL API with `--json`/`--jq` parsing. Triggers on: "create an issue", "submit a PR", "check CI status", "why did CI fail", "merge a PR", or pasted GitHub URLs. |
| **metadata** | <table><tr><th>version</th><th>category</th><th>tags</th><th>difficulty</th></tr><tr><td>1.1.1</td><td>review</td><td><code>github</code> <code>cli</code> <code>issues</code> <code>pull-requests</code></td><td>intermediate</td></tr></table> |

# Git & GitHub Standards

This document defines the core standards for version control, collaboration workflows, and GitHub operations using the `gh` CLI.

---

## 1. Branching Strategy
We follow a **Feature Branch Workflow** to keep the `main` branch stable and deployable at all times.

| Branch Type | Naming Convention | Description |
| :--- | :--- | :--- |
| **Main** | `main` | Production-ready code. No direct commits allowed. |
| **Feature** | `feature/<name>` | New features or enhancements. |
| **Bugfix** | `fix/<name>` | Bug fixes for existing features. |
| **Hotfix** | `hotfix/<name>` | Critical production fixes. |
| **Docs** | `docs/<name>` | Documentation only changes. |

> [!TIP]
> Always create branches from the latest `main`:
> `git checkout main && git pull origin main && git checkout -b feature/my-new-feature`

---

## 2. Commit Conventions
We use **Conventional Commits** to ensure a readable and searchable project history.

**Format:** `<type>(<scope>): <description>`

*   **`feat`**: A new feature
*   **`fix`**: A bug fix
*   **`docs`**: Documentation only changes
*   **`style`**: Changes that do not affect the meaning of the code (white-space, formatting, etc.)
*   **`refactor`**: A code change that neither fixes a bug nor adds a feature
*   **`perf`**: A code change that improves performance
*   **`test`**: Adding missing tests or correcting existing tests
*   **`chore`**: Changes to the build process or auxiliary tools and libraries

**Example:**
`feat(auth): add jwt token validation`

---

## 3. Pull Request (PR) Rules
1.  **Atomicity**: One PR per feature or bug fix. Keep them small.
2.  **Descriptions**: Every PR must include a summary of changes and a link to the related issue.
3.  **Review**: At least one approval is required before merging.
4.  **CI/CD**: All status checks must pass (Linting, Tests, Build).
5.  **Cleanup**: Delete the feature branch immediately after merging.

---

## 4. .gitignore Standards
Every project must include a `.gitignore` file to prevent sensitive or unnecessary files from being tracked.

**Essential Exclusions:**
*   **Secrets**: `.env`, `*.pem`, `creds.json`
*   **Dependencies**: `node_modules/`, `vendor/`, `venv/`
*   **Build Artifacts**: `dist/`, `build/`, `*.exe`
*   **OS Files**: `.DS_Store`, `Thumbs.db`
*   **IDE Config**: `.vscode/`, `.idea/`

---

## 5. GitHub CLI (`gh`) Operations

All GitHub operations use the `gh` CLI. Prefer `--json` with `--jq` for structured, parseable output.

### Authentication & Setup
```bash
gh auth login           # interactive OAuth login
gh auth status          # check current auth and active account
gh auth refresh --scopes repo,read:org,workflow # Add missing scopes
```

### Reference Routing

| User intent | Load |
| :--- | :--- |
| Create/list/view/edit/close issues | `gh issue [list/view/create/edit/close]` |
| Create/list/view/review/merge PRs | `gh pr [list/view/create/review/merge]` |
| List runs, view logs, watch workflows | `gh run [list/view/watch]` |
| Create/fork/clone/view repos | `gh repo [create/fork/clone]` |
| Create/list/view/edit/delete gists | `gh gist [create/list/view/edit/delete]` |

### Calibration Rules

1.  **Prefer `--json` over text output**: Text formats are unstable across versions.
2.  **Use `--jq` to minimize output**: Filter at the source to reduce data transfer.
3.  **Prefer high-level commands**: Use `gh issue create` instead of `gh api` where possible.
4.  **Use `--repo` explicitly**: Never rely on implicit detection in scripts or CI.

### Error Handling

| Error | Cause | Resolution |
| :--- | :--- | :--- |
| **401 Unauthorized** | Token expired | Run `gh auth login` |
| **403 Forbidden** | Missing scopes | Run `gh auth refresh --scopes ...` |
| **404 Not Found** | Repo/Resource missing | Check repo name and permissions |
| **429 Too Many Requests** | Rate limited | Wait or reduce request frequency |

---

## Limitations
*   **Network required**: No offline mode for `gh` commands.
*   **Secrets are write-only**: `gh secret set` works, but you cannot "get" them back.
*   **Artifact retention**: Workflow artifacts expire after 90 days by default.
