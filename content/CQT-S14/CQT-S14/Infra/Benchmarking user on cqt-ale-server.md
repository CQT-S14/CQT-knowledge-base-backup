
[Infra](../Infra.md)

# Benchmarking user on cqt-ale-server

# Benchmarking user setup

## Purpose

The `benchmarking` user on `cqt-ale-server` (54.251.1.211) is the dedicated account that runs the reporting pipeline: it polls the AWS Postgres DB for new experiment results, generates `report.pdf`, and pushes it to the `gh-pages` branch of `CQT-reporting` . This page documents how the user was created and how it authenticates to GitHub.

## User creation

All commands below are run as your own user (e.g. `elis`) on `cqt-ale-server`. You need `sudo` privileges.

```java
# Create the benchmarking user with a home directory and bash shell
sudo useradd -m -s /bin/bash benchmarking

# Set up SSH directory and authorized_keys with correct permissions
sudo mkdir -p /home/benchmarking/.ssh
sudo chmod 700 /home/benchmarking/.ssh
sudo touch /home/benchmarking/.ssh/authorized_keys
sudo chmod 600 /home/benchmarking/.ssh/authorized_keys
sudo chown -R benchmarking:benchmarking /home/benchmarking/.ssh
```

## Granting SSH access

Since multiple people may need to connect as `benchmarking`, each person's public key is added to its `authorized_keys` file (one key per line).

To add your own key (copies it from your current user):

```java
cat ~/.ssh/authorized_keys | sudo tee -a /home/benchmarking/.ssh/authorized_keys
```

When another person needs access, they send you their public key and you run:

```java
echo "their-public-key-here" | sudo tee -a /home/benchmarking/.ssh/authorized_keys
```

To test from your local machine:

```java
ssh benchmarking@54.251.1.211
```

## GitHub authentication via PAT

The `benchmarking` user pushes `report.pdf` to the `gh-pages` branch of the `CQT-reporting` repo. Since this is done programmatically (no interactive login), it uses a GitHub Personal Access Token (PAT) stored in an environment file.

The token lives at:

```java
/home/benchmarking/.env_user
```

Contents:

```java
GITHUB_TOKEN=github_pat_<your_token_here>
```

Scripts source this file and use the token for authenticated pushes:

```java
source /home/benchmarking/.env_user
git push https://${GITHUB_TOKEN}@github.com/CQT-S14/CQT-reporting.git gh-pages
```

This approach is used because `CQT-reporting` is a public repo, so `git clone` works without credentials, but pushing to `gh-pages` requires authentication. A PAT avoids the need for deploy keys or SSH-based GitHub auth on a shared service account.

## Notes

- The PAT must have at least `repo` scope (or fine-grained `contents: write` on `CQT-reporting`).
- If the PAT expires or is revoked, the reporting pipeline will fail at the push step. Regenerate the token in GitHub and update `/home/benchmarking/.env_user`.
- The `.env_user` file should be readable only by the `benchmarking` user (`chmod 600`).
