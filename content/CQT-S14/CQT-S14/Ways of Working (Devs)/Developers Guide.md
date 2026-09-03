
[Ways of Working (Devs)](../Ways%20of%20Working%20(Devs).md)

# Developers Guide

# Working on a feature or fix

1. Create a branch in the relevant repo.

```java
cd <repo-A>
git checkout -b yourname-what-description #eg. eo-bug-2qubit_fidelities
```

2. Develop, test, commit, push.

```java
git add .
git commit -m "description of change"
git push origin eo-bug-2qubit_fidelities
```

3. Open a Pull Request on GitHub to merge into branch `develop`
4. Another team member reviews and approves the PR.
5. Merge.

# Picking up someone else's merged changes

Because repos are installed in editable mode, pulling new code is all you need to have updated code become immediately active in your environment.

**Important: pull all repos, not just one.** Since your environment runs whatever code is checked out locally in each clone, it's important to keep branch `develop` pulled in all three repos.

So, before you start a development session, check out `develop` in each repo and pull:

```java
git checkout develop && git pull
```

> [!TIP]
> Do this for all forked repos to ensure your development workspace env is consistent and up to date.

> [!WARNING]
> **Reminder:** If you are working on a different branch in any repo, commit or stash your uncommitted changes before checking out to `develop`. Otherwise Git will refuse to switch branches, or worse, your in-progress work may be lost.

# When a change affects dependencies

There are two cases:

**a) A repo needs a new or different library** (e.g., adding scipy, changing a numpy version): this goes in that repo's own dependency spec (`pyproject.toml` / `poetry.lock` / `requirements.txt`). The PR should clearly state what was added or changed and why.

After any dependency changes happen you’ll need to recreate the workspace environment:

```java
cd ./envs
rm -r workspace_env_CQT-experiments

cd CQT-experiments/workspaces
bash setup_dev_env.sh

```

**b) A new release of one forked repo breaks compatibility with another**: update the compatibility matrix in CQT-experiments to pin whichever is the last working combination. This also goes through a PR.

**c) A new custom driver or changes in Keysight’s drivers:**

Ideally there is only one source o truth for the environment `keysight-qcs-py312`. That means if it changed your copy in folder `/envs` need to be re-cloned:

```java
# If virtualenv-clone is not pre-installed
pip3 install virtualenv-clone # if 

# If it installed successfully, but the `virtualenv-clone` command won't work
yet because `~/.local/bin` isn't in your `PATH`. Quick fix:

export PATH="$PATH:/mnt/home/eortiz/.local/bin"

# Then clone
virtualenv-clone /mnt/scratch/envs/keysight-qcs-py312 ~/envs/keysight-qcs-py312
```
