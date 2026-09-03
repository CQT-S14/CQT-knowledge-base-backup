
[Infra](../Infra.md)

# Workspace Environment

- [What Is the Workspace Environment?](#workspaceenvironment-whatistheworkspaceenvironment)
- [What It Does](#workspaceenvironment-whatitdoes)
- [How setup\_dev\_env\_v2.sh Works (Step by Step)](#workspaceenvironment-howsetup-dev-env-v2shworksstepbystep)
  - [Step 1: Creates an Empty Python Environment](#workspaceenvironment-step1createsanemptypythonenvironment)
  - [Step 2: Installs Pinned Packages](#workspaceenvironment-step2installspinnedpackages)
  - [Step 3: Installs Editable Packages](#workspaceenvironment-step3installseditablepackages)
  - [Step 4: Adds the Hardware Fallback Path](#workspaceenvironment-step4addsthehardwarefallbackpath)
- [Python's Import Search Order and Shadowing](#workspaceenvironment-pythonsimportsearchorderandshadowing)
  - [Example: keysight (Hardware Drivers)](#workspaceenvironment-examplekeysighthardwaredrivers)
- [Determining Which Versions You're Using](#workspaceenvironment-determiningwhichversionsyoureusing)
- [When to Rebuild the Workspace](#workspaceenvironment-whentorebuildtheworkspace)

- [Caution note](#workspaceenvironment-cautionnote)
  - [Summary: Key Points](#workspaceenvironment-summarykeypoints)

## What Is the Workspace Environment?

A Python virtual environment (`venv`) created at `~/envs/workspace_env_{local name of your cloned CQT-experiments repo}` that contains:

- A Python 3.12 interpreter
- Editable installs of your custom (qibo) packages (CQT-qibocal, CQT-qibolab) which take preference. Ie. when doing "import qibolab" --> you are importing CQT-qibolab.
- Pinned versions of compatible qibo packages (qibo, qibolab, qibocal) for when no local editables are installed.

The workspace environment is isolated from system Python and from other projects' Python environments. The goal is to use workspace environment for launching experiments via slurm and to facilitate development of `qibo` packages in reproducible ways.

## What It Does

1. **Ensures reproducibility** — Everyone running `setup_dev_env.sh` knows which package versions are being installed and used. Qibo packages versions and mount points are explicitly printed out when creating the workspace environment.
2. **Isolates dependencies** — Your project's packages don't conflict with system packages (nor local machine, nor server)
3. **Allows editable installs** — You can modify CQT-qibocal or CQT-qibolab code and changes apply immediately without having to re-create the environemnt. Good for developers.

## How setup\_dev\_env\_v2.sh Works (Step by Step)

### Step 1: Creates an Empty Python Environment

```java
cd CQT-experiments/workspace

bash setup_dev_env.sh
```

Creates a new directory at `~/envs/workspace_env_CQT-experiments/` containing:

- A fresh Python 3.12 executable
- An empty `lib/python3.12/site-packages/` directory (where packages are stored)
- Activation scripts

At this point, the environment has no packages installed—it's blank.

### Step 2: Installs Pinned Packages

Installs exact pinned versions (from the `compatibility_matrix.toml`) of two packages into the workspace venv's site-packages:

- `qibo==0.2.23`
- `qibocal==0.2.5`
- `qibolab==0.2.9`

These versions are known to be compatible with each other.  
In general, `qibo` latest releases should be compatible with each other. However, that's not always the case since different parts of the `qibo` ecosystem advance independently and that's why we pin the latest compatible ones.

Your workspace now contains these two packages in `~/envs/workspace_env_CQT-experiments/lib/python3.12/site-packages/`.

### Step 3: Installs Editable Packages

```java
pip install -e ~/CQT-experiments
pip install -e ~/CQT-qibocal
pip install -e ~/CQT-qibolab
```

The `-e` flag means **editable install**. Instead of copying package files into site-packages, pip creates a reference that points to the original local folder.

**How editable packages work**

When you do in your code `import qibocal`, Python looks in `CQT-qibocal` local folder, in the git branch you are currently checked out, and this is what gets imported and used.

Specifically,  
CQT-qibolab's `pyproject.toml` declares `name = "qibolab"` (same package name)

- pip recognizes this as a replacement for the PyPI `qibolab`
- pip creates `qibolab.pth` (or `direct_url.json`) that redirects imports of `qibolab` to `~/CQT-qibolab/`

If CQT-qibolab's `pyproject.toml` declared instead `name = "CQT-qibolab"` then `import qibolab` will import the PyPI `qibolab==0.2.9` earlier installed and `import CQT-qibolab` would import as a separate package the local editable `CQT-qibolab`.

**Why editable?**

- You can edit the code in `~/CQT-qibocal` and changes take effect immediately
- You don't need to reinstall the environment after every code change

**What also gets installed:**  
Each of your packages has a `pyproject.toml` that lists its dependencies. When pip installs your editable packages, it also installs all their dependencies. For example:

- `CQT-qibolab` depends on `matplotlib>=3.8.0`
- pip resolves this to a compatible version (e.g., `matplotlib==3.8.5`) and installs it to your workspace env.
- `CQT-qibocal` depends on `numpy>=1.20.0`
- pip installs the resolved version (e.g., `numpy==1.24.0`)

**Result:** Your venv now contains:

- CQT-experiments, CQT-qibocal, CQT-qibolab (local editable packages that take preference)
- `qibo==0.2.23` and `qibolab==0.2.9` (from step 2, which will get only be used if no editables overtake them)
- All transitive dependencies (matplotlib, numpy, pandas, scipy, etc.) resolved and installed wihtout conflicts.

### Step 4: Adds the Hardware Fallback Path

```java
echo "/mnt/home/eortiz/envs/keysight-qcs-py312/lib/python3.12/site-packages" \

> ~/envs/workspace_env_CQT-experiments/lib/python3.12/site-packages/zzz_keysight_fallback.pth

```

Creates a `.pth` file (a text file that modifies Python's import path) that appends `keysight-qcs-py312`'s site-packages to `sys.path` (Python's package search list).

The goal of this is to install every other hardware driver bit that is needed to run experiments in the cluster (eg. keysight's drivers). These packages are to be found in a specifically designated environment available in the `CQT-S14 nqch` server at: `/mnt/scratch/envs/keysight-qcs-py312`.

**Note**

For `setup_dev_env.sh` to work you need to clone this into your user `/env` folder first:

```java
pip3 install virtualenv-clone
virtualenv-clone /mnt/scratch/envs/keysight-qcs-py312

```

Otherwise `setup_dev_env.sh` won't be able to reference it and find and install the necessary hardware packages from there.

## Python's Import Search Order and Shadowing

When you run `import qibolab` or `import matplotlib`, Python searches through `sys.path` in order and uses the **first match it finds**. It does not continue searching after finding a match.

After setup\_dev\_env\_v2.sh completes, `sys.path` is searched in this order:

1. `~/envs/workspace_env_CQT-experiments/lib/python3.12/site-packages/` (your venv)
2. `~/envs/keysight-qcs-py312/lib/python3.12/site-packages/` (keysight fallback)

If a package is found in location 1, location 2 is never checked. We say location 1 "shadows" location 2.

This is important, because it means even if redundant packages (eg. matplotlib) exist in `keysight-qcs-py312` those won't be picked.

### Example: keysight (Hardware Drivers)

keysight is not in your editable packages' dependencies. No step above installs it into your venv.

**Installation sequence:**

```java
# Steps 1-3: nothing installs keysight into your venv
# Step 4: .pth file adds keysight-qcs-py312 to sys.path
```

**When you** `import keysight`**:**

- Python searches location 1 (your venv): NOT FOUND
- Python searches location 2 (keysight-qcs-py312): FOUND
- **You get** `keysight-qcs==2.6.4` from the fallback

This is why keysight-qcs-py312 is called a **fallback**—it's only used for packages not in your venv.

## Determining Which Versions You're Using

All packages are in `~/envs/workspace_env_CQT-experiments/lib/python3.12/site-packages/`.

To find what version of a package is active:

```java
source ~/envs/workspace_env_CQT-experiments/bin/activate
pip show <package_name>
```

To find where an editable package is located:

```java
pip show qibocal
```

To check if something is shadowing something else:

```java
python3 -c "import sys; print('\n'.join(sys.path))"
```

This prints Python's import search order. Your venv's site-packages should appear before keysight-qcs-py312.

## When to Rebuild the Workspace

**Delete workspace\_env\_CQT-experiments and Re-Run setup\_dev\_env.sh again if:**

rebuild: `bash setup_dev_env.sh`

1. If `~/CQT-qibocal` or `~/CQT-qibolab` have new repo dependencies (ie. updated .toml)
2. You updated `compatibility_matrix.toml`
3. Something in your environment is broken and you want a clean rebuild
4. The HPC admin confirms `keysight-qcs-py312` has changed

**Do NOT rebuild if:**

1. You edited code in `~/CQT-experiments`, `~/CQT-qibocal`, or `~/CQT-qibolab` — editable installs apply changes immediately

# Caution note

> [!WARNING]
> If you run `setup_dev_env.sh` from a parent like `forks-development/CQT-experiments`, it creates an environment folder called `~/envs/workspace_env_CQT-experiments`. If you then run it from another parent's clone, e.g. `forks-benchmarking-pipeline/CQT-experiments` it creates a folder with the **same name**, `~/envs/workspace_env_CQT-experiments`, **so it overwrites the previous one.**

That means when you launch Slurm jobs from either `forks-development/CQT-experiments` or `forks-benchmarking-pipeline/CQT-experiments`, both use the **last** `workspace_env_CQT-experiments` that was built. This may not be the correct/compatible environment for `forks-benchmarking-pipeline/CQT-experiments` (or for whichever clone you didn't build last).

> [!TIP]
> **Fix:** after building, rename the environment per clone (e.g. `workspace_env_development`, `workspace_env_benchmarking_pipeline`) so they don't overwrite each other.

## Summary: Key Points

1. **Your venv is isolated**: It contains specific versions, independent from system Python
2. **Editable packages override pinned versions**:When you `import qibolab`, you get `~/CQT-qibolab`, not PyPI's version
3. **Dependencies are resolved at install time**: matplotlib, numpy, etc. are pinned to specific compatible versions resolved by pip
4. **Your venv's packages shadow keysight-qcs-py312**: Anything in your venv is used first; only keysight drivers and alike come from the fallback
5. **Edits apply immediately**: No environment reinstall needed for editable packages.
