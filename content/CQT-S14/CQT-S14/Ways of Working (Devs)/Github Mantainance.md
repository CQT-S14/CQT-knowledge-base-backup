
[Ways of Working (Devs)](../Ways%20of%20Working%20(Devs).md)

# Github Mantainance

## Weekly Maintenance (single person, same cadence)

### 1. Promote `develop` → `stable-pipeline`

> [!IMPORTANT]
> This is needed to guarantee that the automated benchmarking experiments are run with the most current stable code our team uses to run experiments.

For each of the three repos:

1. `git checkout stable-pipeline`
2. `git merge develop` (should be a fast-forward)
3. **If changes in** `develop` **introduced new dependencies or version changes:** regenerate the pipeline's virtual environment for `forks-benchmarking-pipeline/` before testing.
4. Confirm the three repos work together (smoke test: imports resolve, a trivial experiment submits correctly).
5. If clean: `git push origin stable-pipeline`
6. If broken: `git reset --hard origin/stable-pipeline`, investigate, and defer promotion.

Then, in the `forks-benchmarking-pipeline/` directory on the server, pull `stable-pipeline` for each repo.

### 2. Sync `main` with upstream

> [!IMPORTANT]
> This is needed so that we can incorporate updates from qiboteam and Scinawa repositories. We must decide to adopt updates and pass them down to our `develop` branch only after testing they don’t brake our development workspace.

This process uses a dedicated `sync` branch per repo and a dedicated virtual environment (`workspace_env_sync_test`) to verify upstream changes before they touch `main`.

For each forked repo:

1. Check out to branch `sync` : `git checkout sync`
2. Pull the upstream main changes into the `sync` branch:

   ```java
    git fetch upstream && git merge upstream/main
   ```

Once all repos are checked out on their `sync` branches and have the upstream changes:

4. Regenerate the dedicated test environment `workspace_env_sync_test` by doing:

   ```java
   cd CQT-experiments/workspace
   bash setup_dev_env_sync_test.sh 
   #same as setup_dev_env.sh with suffix _sync_test for the workspace_env_{} folder
   ```
5. If everything passes: for each repo, fast-forward `main` to match upstream

   ```java
   # In each repo:
   git checkout main && git merge upstream/main && git push origin main.
   ```
6. If anything fails: do not update `main`. Document what broke and discuss in the next meeting.

> [!WARNING]
> **Note:** A successful upstream sync to `main` does not automatically flow into `develop`. **Merging** `main` **into** `develop` **is a separate, deliberate decision** made when the team wants to incorporate upstream changes into the working state. Communicate with the team and agree a to do it.
