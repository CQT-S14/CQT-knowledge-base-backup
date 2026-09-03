
[Ways of Working (Devs)](../Ways%20of%20Working%20(Devs).md)

# Important Rules

## 🛑 Rules (non-negotiable)

1. **Never install anything into the workspace\_env\_CQT-experiments manually.**   
   Nobody should ever `source activate workspace_env_CQT-experiments` and then `pip install` directly into it. The workspace should only ever be populated by the script. If someone pip-installs into the active workspace, it becomes an untracked change that nobody else has. Do not do this. Instead, follow the correct process for adding a dependency:

   - Edit the repo's `pyproject.toml` (or `setup.cfg` / `requirements.txt`) to add the new package.
   - Regenerate the lockfile (e.g., `poetry lock` if using Poetry) so versions are resolved and recorded.
   - Delete and recreate the workspace by rerunning `setup_dev_env.sh`.
   - Commit both the updated `pyproject.toml` and the lockfile, push, and open a PR.

   The rule is really:   
   **Only add dependencies through the repo's source files → rebuild workspace → PR.**  
   **The workspace is always a derived artifact, never manually modified.**
2. **Everyone uses the same workspace script.**   
   No personal environment tweaks. If changes to the scripts inside folder `/workspace` are needed, create a feature branch, modify stuff and open a PR to branch `develop`.
3. **Unpushed "forever local" branches are strictly forbidden.**   
   All code goes through GitHub. Branches → Pull Requests → review → merge. No code lives for long time only on your local machine or any unpushed branch for long.

## ⚠️ Important note on editable mode

Editable mode means your environment always runs whatever code is in your local clones, including uncommitted or unpushed changes. This is powerful but comes with a responsibility:

**If you change code locally and don't push it, nobody else has it.** Your environment works, theirs doesn't, and we're back to "works on my machine."

This means:

- **Never sit on local-only changes that others depend on.** If your change affects another repo or another person's work, push it and open a PR promptly.
- **No permanent local branches.** Every branch should exist on GitHub CQT-S14. If it's not pushed, it doesn't exist as far as the team is concerned.
- **Commit and push frequently.** Small, regular pushes are better than large, infrequent ones. This keeps everyone in sync and makes reviews easier.
- **Stay on** `develop` **for repos you're not actively developing features on.** Branch `develop` is the working state of our team for each repo. If you're working on repo A but still checked out on an old branch in repo B, your environment runs stale repo B code. Keep your `develop`(s) pulled and current.

Editable mode saves us from constantly rebuilding the environment. In return, we commit to keeping our local clones clean and pushed. That's the trade-off.

# 🥷 Syncing forks with qiboteam (Upstream)

Our forked repos need to stay in sync with their upstream repositories to pick up bug fixes, new features, and releases from the open-source projects.

**Who:** One designated person is responsible for syncing at any given time. [Github Mantainance](Github%20Mantainance.md) This can rotate, but there must always be a clear owner so it doesn't get forgotten.

**When:** On a regular cadence (e.g., weekly), or on-demand when someone notices a relevant upstream release.

**How:** Upstream changes can break things — a new upstream release might introduce changes that our other repos or pinned dependencies aren't ready or can support. For this reason, never sync `upstream` directly into your cloned fork's `develop` nor `main`. **Ask the person responsible for syncing with upstream to incorporate the upstream changes into the fork.**

> [!WARNING]
> This ensures upstream updates never break the team's environment.
