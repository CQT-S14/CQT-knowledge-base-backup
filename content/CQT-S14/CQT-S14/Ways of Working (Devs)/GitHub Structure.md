
[Ways of Working (Devs)](../Ways%20of%20Working%20(Devs).md)

# GitHub Structure

# GitHub Structure

## Branch Model (same across all 3 fork repos)

| <mark style="background: #ff8f73;">**Branch**</mark> | <mark style="background: #ff8f73;">**Purpose**</mark> | <mark style="background: #ff8f73;">**Updated by**</mark> |
| --- | --- | --- |
| `main` | Mirror of upstream. Never commit directly. | Weekly. Manual sync with upstream |
| `develop` | **Team's working state.** All active development merges here. | PRs from feature/fix branches |
| `stable-pipeline` | Stable snapshot of `develop` used by the benchmarking pipeline. | Weekly fast-forward merge from `develop` (manual) |
| `sync` | Temporary branch for testing upstream changes before updating `main`. | Created from `upstream/main` during weekly sync process |

# Server Directory Layout

### `forks-development/`

Contains clones of the 3 org fork repos. This is where team members develop.

- All three repos are installed as editable packages in virtual environment `envs/workspace_env_CQT-experiments`
- Checkout from `develop` branch to start developing / running experiments or anything.
- Developers create feature branches off `develop`, then open PRs back to `develop` to keep our team working state current for us.
- `main` exists locally but you should ignore it and not use at all. It is only for upstream synchronization.

### `forks-benchmarking-pipeline/`

Contains separate clones of the same 3 org fork repos. This is exclusively for running the automated benchmarking pipeline to produce automated reports. It is mainly hands-off do not touch.

- All three repos are installed as editable packages in virtual environment `envs/workspace_env_CQT-experiments_benchmarking-pipeline` dedicated to the pipeline.
- The 3 repos are permanently checked out to branch: `stable-pipeline`
- No one develops here. Do not switch branches or make local changes.

# Rules

- **Never commit directly to** `main` **or** `stable-pipeline`**.** Both are downstream mirrors.
- **All development goes through** `develop` **via PRs.** Even small patches.
- `forks-benchmarking-pipeline/` **is hands-off.** No local edits.
- **If onboarding a new team member:** clone repos, checkout `develop`, start working.
