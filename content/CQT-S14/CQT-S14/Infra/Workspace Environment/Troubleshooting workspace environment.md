
[Infra](../../Infra.md) > [Workspace Environment](../Workspace%20Environment.md)

# Troubleshooting workspace environment

- [How does the workspace environment work?](#troubleshootingworkspaceenvironment-howdoestheworkspaceenvironmentwork)
- [Troubleshooting](#troubleshootingworkspaceenvironment-troubleshooting)
  - [Q: Which environment am I am currently using?](#troubleshootingworkspaceenvironment-qwhichenvironmentamiamcurrentlyusing)
  - [Q: My experiment uses the wrong version of qibolab. How do I fix it?](#troubleshootingworkspaceenvironment-qmyexperimentusesthewrongversionofqibolabhowdoifixit)
  - [Q: keysight-qcs-py312 was updated and my code breaks. What do I do?](#troubleshootingworkspaceenvironment-qkeysight-qcs-py312wasupdatedandmycodebreakswhatdoido)
  - [Q: Why does my edited code not take effect?](#troubleshootingworkspaceenvironment-qwhydoesmyeditedcodenottakeeffect)
  - [Q: What if I want to use a different version of qibo-sth?](#troubleshootingworkspaceenvironment-qwhatifiwanttouseadifferentversionofqibo-sth)

# How does the workspace environment work?

You can read the full details in [Workspace Environment](../Workspace%20Environment.md)section.

The crucial aspects to understand are that the script that sets up the workspace environment, does the following:

1. It Installs sibling cloned repos in **editable mode**, so code changes are picked up instantly by the virtual environment, no reinstall/rebuild needed.

> [!NOTE]
> *Note: The repos installed in editable mode are CQT-experiments, CQT-qibocal and CQT-qibolab, which are currently the ones specified in setup\_dev\_env.sh. Repos to be installed in editable mode need to pre-exist cloned in the same directory as CQT-experiments (thus why called siblings)*

2. It Pins cross-repo versions using a **compatibility matrix** so everyone uses a combination of repo versions known to work together.

> [!NOTE]
> *Note: the compatibility matrix only tracks which versions of the repos are compatible with each other. Individual package dependencies (numpy, scipy, etc.) are managed by each repo's own dependency spec (*`pyproject.toml`*,* `poetry.lock`*, or* `requirements.txt`*).*

3. Installs hardware drivers (CUDA, Keysight) from a pre-existing virtual environment `keysight-qcs-py312.` In previous versions it got these things via `module load qibo`.

> [!NOTE]
> *Note: we are migrating away from* `module load qibo`*. The goal is a self contained environment or module containing only drivers and hardware interfaces — nothing that can be pip-installed. In a beautiful world ideally this layer gets dockerized and versioned too…*

# Troubleshooting

### Q: Which environment am I am currently using?

Run:

```java
which python
echo $VIRTUAL_ENV
```

Should output: `~/envs/workspace_env_CQT-experiments`

### Q: My experiment uses the wrong version of qibolab. How do I fix it?

Check what's installed:

```java
pip show qibolab
```

If `Location` shows a site-packages directory instead of `~/CQT-qibolab`, the editable install failed. Reinstall:

```java
pip install -e ~/CQT-qibolab
```

### Q: keysight-qcs-py312 was updated and my code breaks. What do I do?

`keysight-qcs-py312` is a fallback—only packages not in your venv are used from it. If your venv has its own version, `keysight-qcs-py312`'s update doesn't affect you.

If the keysight hardware drivers themselves changed:

1. Ask the HPC admin (Paul) what changed
2. Verify your venv still works: `pip list | grep keysight`
3. If something is truly broken, rebuild: `bash setup_dev_env.sh`

### Q: Why does my edited code not take effect?

If you edited `~/CQT-qibolab` but changes don't appear:

1. Confirm the editable install exists: `pip show qibocal` should show editable location
2. Confirm you're using the activated venv: `echo $VIRTUAL_ENV`
3. Restart your Python process (edits to source files take effect only on next import)

### Q: What if I want to use a different version of qibo-sth?

1. Create a separate git branch and then
2. Update `compatibility_matrix.toml` with your new versions
3. Remove your current workspace*env*{repo-name} if you plan to re-create it from the same repo
4. Run `bash setup_dev_env.sh` again
5. This will pin the new qibo versions and reinstall everything
6. Notice that new releases and versions are not guaranteed to work with the rest of the repos just because you add them to the `compatibility_matrix.toml.` `compatibility_matrix.toml` is not a magic file that makes things work, it simply states a combination of versions that has been tested and is known to work. If you add different versions they may or may not be compatible together. If they are you'll get the workspace environment setup up without problem, if they don't you'll see the process crash.  
   It is interesting to test new version out in a separate branch so that if you find a combo that works you can directly push the new `compatibility_matrix.toml` to the `CQT-experiments (S14 fork)` and open a PR.
