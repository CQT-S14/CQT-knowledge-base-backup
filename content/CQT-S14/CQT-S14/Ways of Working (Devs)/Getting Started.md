
[Ways of Working (Devs)](../Ways%20of%20Working%20(Devs).md)

# Getting Started

# Setting up a Workspace Environment

#### 1. Create a folder structure like this:

```java
# If in nqch server, for any user
your_user_name # if in the server | or your local projects folder if locally
  |_ envs/
  |_ forks-development/
  |_ .env_user

# If in your local machine, in your project's folder
quantum_projects
  |_ envs/
  |_ forks-development
  |_ .env_user
```

In `/envs` is where we’ll generate our workspace environment, which will give us everything we need to work (both locally and in the server).

There is a difference between setting things up in the server versus your local machine though. This is because from the server we can launch actual experiments against the quantum computer, thus we need extra things to run the hardware. In the server, before we generate our workspace environ we’ll clone another environment named `keysight-qcs-py312` that contains mainly only hardware related installs that our workspace will then be able to use.

To clone the virtual environment `keysight-qcs-py312` into your `envs` folder do:

```java
pip3 install virtualenv-clone
virtualenv-clone /mnt/scratch/envs/keysight-qcs-py312 keysight-qcs-py312

# If you get a PATH warning message do this
export PATH="$PATH:/mnt/home/nqch-deploy/.local/bin" # substitute nqch-deploy for the correct username
```

You now have the following structure in the server:

```java
# In nqch server
your_user_name # if in the server | or your local projects folder if locally
  |_ envs/
  |   |_ keysight-qcs-py312
  |_ forks-development/
  |_ .env_user
```

#### 1. Clone forked repositories from CQT-S14 Github Organization

From here onwards all steps are the same for setting up things in the server vs locally so we’ll only demonstrate assuming you are in an nqch server user.

The next thing to do is to clone the repos from CQT-S14 org, which are the forks our team will do development against.

```java
cd forks-development

# Then clone: 
https://github.com/CQT-S14/CQT-qibolab.git
https://github.com/CQT-S14/CQT-qibocal.git
https://github.com/CQT-S14/CQT-experiments.git
# ... any additional CQT-S14 repos
```

**3. Get remote branches and leave the repos checked out in branch** `develop`

In order to create the workspace environment that uses the package-repos just cloned in editable mode, *in the state that corresponds to the team stable development environment*, you need to be checked out in branch `develop` in every repo before you create the environment.

For that we 1st need to “obtain” locally these branches.

```java
# Goto 1st repo from set of CQT-experiments, CQT-qibocal, CQT-qibolab
cd CQT-experiments
git branch                   # check you are in branch main 
git checkout main            # if for whatever reason you are not in main
git fetch                    # to fetch remotes
git branch -a                # to check the remotes have been fetched

git checkout -b sync origin/sync          # get local sync branch
git checkout -b develop origin/develop    # get local develop branch

# move on to the next repo and do the same
cd ../CQT-qibocal
git checkout main 
git fetch 
git checkout -b sync origin/sync          # get local sync branch
git checkout -b develop origin/develop    # get local develop branch

# and finally repeat for the last repo
cd ../CQT-qibolab
git checkout main 
git fetch
git checkout -b sync origin/sync          # get local sync branch
git checkout -b develop origin/develop    # get local develop branch
```

Make sure you are checked out in branch `develop` in all 3 repos at the end before next step

#### 4. Create the workspace environment

We’ll run the script that sets up the workspace environment from repo `CQT-experiments/workspace`. This script will create a virtual environment under name `workspace_env_CQT-experiments`

```java
cd forks-development/CQT-experiments/workspace
bash setup_dev_env.sh
```

That's it. You're ready to develop.   
The next step is to check out the -> [Developers Guide](Developers%20Guide.md)
