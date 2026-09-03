
[Infra](../../Infra.md) > [Servers](../Servers.md)

# Access nqch server (new users)

# Login new users

## Quick steps to nqch (paul) server access

**Step 1 — Find or create your SSH public key**  
Check what keys you already have:

```java
ls ~/.ssh/*.pub
```

If you see a file listed (e.g. `id_ed25519.pub`, `id_rsa.pub`, or similar), display it:

```java
cat ~/.ssh/id_ed25519.pub
```

(replace the filename with whatever showed up in the `ls` command above)  
If `ls` says "No such file or directory", you don't have a key yet — create one:

```java
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Press Enter to accept all default prompts (including empty passphrase, unless you want one). Then run the `cat` command above to display your new key.

**Step 2 — Send me that key**  
Copy the full output (starts with `ssh-ed25519` or `ssh-rsa`, ends with your email/comment) and send it to me. I'll add it to the server.

**Step 3 — Wait for my confirmation**, then test:

```java
ssh username@10.246.80.228
```

You should log in without being asked for a password. If it works, continue to Step 4.

**Step 4 — (Optional but recommended) Add a shortcut to your SSH config**  
Open your SSH config file:

```java
nano ~/.ssh/config
```

Add this block at the end (paste it exactly):

```java
# Paul server for CQT Qibo user mustafa
Host cqt-paul-server-username
    HostName 10.246.80.228
    User username

```

Save and exit (in `nano`: Ctrl+O, Enter, Ctrl+X).

From now on, you can just type:

```java
ssh cqt-paul-server-username
```

instead of the longer `ssh username@10.246.80.228` — both will work identically.

# Onboarding

Repeatable runbook for getting a brand-new (or new-team-member) checkout ready to submit experiment jobs to the nqch quantum computer via Slurm on `cqt-paul-server`. Follow the steps in order, top to bottom.

## Prerequisites

- SSH access to `cqt-paul-server` (`10.246.80.228`) — requires NUS VPN.
- You're logged in and sitting in your server user home directory (the root that will hold `forks-CQT-S14/`, `envs/`, and `.env_user` as siblings).

## Step 1 — Create the folder skeleton

`envs/` and `forks-CQT-S14/` sit **next to each other** at your user home root — `envs/` is not nested inside `forks-CQT-S14/`.

```java
mkdir -p envs forks-CQT-S14
```

Note: the `.env_user` file (pasted in manually — not covered by this doc) also goes at this same root level, alongside `envs/` and `forks-CQT-S14/`.

## Step 2 — Clone the hardware driver env (server-only step)

Because this server can launch real experiments against the quantum computer, the workspace environment needs Keysight hardware drivers available. Clone the shared driver env into `envs/` (at your home root) before generating your workspace env:

```java
cd envs
pip3 install virtualenv-clone
virtualenv-clone /mnt/scratch/envs/keysight-qcs-py312-274 keysight-qcs-py312-274
```

If `virtualenv-clone` isn't found on your PATH:

```bash
export PATH="$PATH:/mnt/home/<your-username>/.local/bin"
```

Then move into `forks-CQT-S14/` for the next steps:

```java
cd ../forks-CQT-S14
```

## Step 3 — Clone the 3 forked repos

```java
git clone https://github.com/CQT-S14/CQT-qibolab.git
git clone https://github.com/CQT-S14/CQT-qibocal.git
git clone https://github.com/CQT-S14/CQT-experiments.git
```

## Step 4 — Check out `main`, `sync`, `develop` locally in each repo, end on `develop`

Repeat this for **each** of the 3 repos (`CQT-qibolab`, `CQT-qibocal`, `CQT-experiments`):

```java
cd <repo-name>

git checkout main            # confirm/land on main
git fetch
git branch -a                # confirm remote branches (origin/develop, origin/sync) are visible

git checkout -b sync origin/sync
git checkout -b develop origin/develop

cd ..
```

**All 3 repos must end up checked out on** `develop` before moving to the next step — the workspace environment you generate next will reflect whatever is currently checked out.

## Step 5 — Generate the workspace environment

```java
cd CQT-experiments/workspace
bash setup_dev_env.sh
```

What this does:

- Creates a fresh Python 3.12 virtualenv at `envs/workspace_env_CQT-experiments`.
- Installs the pinned, mutually-compatible `qibo`/`qibolab`/`qibocal` versions from `compatibility_matrix.toml`.
- Editable-installs the 3 local repos (`CQT-experiments`, `CQT-qibocal`, `CQT-qibolab`) on top — these take precedence over the pinned versions, so `import qibolab` resolves to your local `CQT-qibolab` clone.
- Wires up `keysight-qcs-py312` (cloned in Step 2) as a fallback on `sys.path`, for hardware driver packages not present in the venv.

## Step 6 — Verify

```bash
source ../../../envs/workspace_env_CQT-experiments/bin/activate   # from CQT-experiments/workspace
which python
echo $VIRTUAL_ENV
pip show qibolab   # Location should point at .../CQT-qibolab (confirms editable install, not the pinned/PyPI version)
```

## Checklist recap

- [ ] `envs/` and `forks-CQT-S14/` created as siblings at your home root
- [ ] `keysight-qcs-py312` cloned into `envs/`
- [ ] `CQT-qibolab`, `CQT-qibocal`, `CQT-experiments` cloned into `forks-CQT-S14/`
- [ ] All 3 repos have local `main`, `sync`, `develop` branches, and are checked out on `develop`
- [ ] `setup_dev_env.sh` run successfully from `CQT-experiments/workspace`, producing `envs/workspace_env_CQT-experiments`
- [ ] Verified: `pip show qibolab` points at your local `CQT-qibolab` clone
- [ ] `.env_user` pasted manually at the home root, alongside `envs/` and `forks-CQT-S14/` (not covered here)

Once all boxes are checked, you're ready to submit Slurm jobs to the nqch cluster.
