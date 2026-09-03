
[Security](../Security.md)

# Leaked credentials

# What to do when credentials are pushed to Git

## 1. Discovering if there was a leak

### Check if a secrets file was ever committed to git history

```java
git log --all --full-history -- .secrets.toml
```

- **If it returns commits**: the file is in your git history — you need to clean it.
- **If it returns nothing**: the file was never committed.

### Check for any references to the secrets file across all commits

```java
git grep "secrets.toml" $(git rev-list --all)
```

- References in `.gitignore`, source code, or logs (e.g. `secrets_path = Path(".secrets.toml")`) are **harmless text references**, not actual secrets.
- If `git log` returned nothing but `git grep` shows results, you're fine — no cleanup needed.

---

## 2. Cleaning the git history

### Prerequisites

```java
pip install git-filter-repo
```

### Step 1: Clone a fresh copy

```java
git clone <repo-url> <repo-name>-clean
cd <repo-name>-clean
```

### Step 2: Remove the file from all history

```java
python3 -m git_filter_repo --path .secrets.toml --invert-paths
```

### Step 3: Re-add your remote (filter-repo removes it)

```java
git remote add origin <repo-url>
```

### Step 4: Force push the cleaned history

```java
git push origin --force --all
```

### Step 5: Prevent future leaks

```java
echo ".secrets.toml" >> .gitignore
git add .gitignore
git commit -m "Add .secrets.toml to gitignore"
git push
```

> **Note:** `.gitignore` only prevents **new** files from being tracked. It does not remove files that are already committed — that's why `git filter-repo` is needed.

### Step 6: Delete old remote branches that still carry dirty history

```java
git push origin --delete <branch-name>
```

### Step 7: Verify the cleanup

```java
git log --all --full-history -- .secrets.toml   # should return nothing
git grep "secrets.toml" $(git rev-list --all)    # text references are fine
```

---

## 3. Cleaning local branches

If you have local branches based on the old (dirty) history that you want to keep, **do not re-run filter-repo on each branch**. Instead, rebase them onto the clean `main`.

### Step 1: Update local main to match the clean remote

```java
git checkout main
git fetch origin
git reset --hard origin/main
```

### Step 2: Rebase each branch onto clean main

```java
git checkout <branch-name>
git rebase main
```

During rebase you may encounter:

- **Conflicts**: open the conflicted files, resolve them (in VS Code use "Accept Incoming Change" to keep your branch's work), then:

  ```java
  git add \<resolved-files\>git rebase --continue
  ```
- **Vim editor opening**: type `:wq` and press Enter to save and continue.
- `.secrets.toml` **sneaking back into staged changes**: the old branch may have a commit that originally added the file. Always check `git status` before continuing and remove it if staged:

  ```java
  git restore --staged .secrets.tomlrm .secrets.tomlgit rebase --continue
  ```

Repeat until you see `Successfully rebased`.

### Step 3: Clean up stale remote references

```java
git fetch --prune
```

If you have an `upstream` remote you no longer need:

```java
git remote remove upstream
```

### Step 4: Final verification across all branches

```java
git log --all --full-history -- .secrets.toml
```

Must return nothing. If it still shows commits, a local or remote branch still has dirty history — identify it with `git branch -a` and delete or rebase it.

### Step 5: Push the cleaned branches

```java
git push -u origin <branch-name>
```

---

## 4. Critical reminder

**Rotate all your secrets immediately.** Removing the file from git history does not undo the exposure. Any credentials that were in the file should be considered compromised — change all passwords, API keys, database URIs, and tokens.
