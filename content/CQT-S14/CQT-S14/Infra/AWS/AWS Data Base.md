
[Infra](../../Infra.md) > [AWS](../AWS.md)

# AWS Data Base

# Benchmarking Database (Tanvirul’s qibodb)

## Overview

The benchmarking database stores calibration data and experiment results produced by the benchmarking pipeline. It is a PostgreSQL database hosted on AWS RDS, accessed through a Flask API layer running on a dedicated EC2 instance. The database is commonly referred to as "Tanvirul's DB".

## Database (RDS)

The database is a managed PostgreSQL instance on AWS RDS in the `ap-southeast-1` region.

| Property | Value |
| --- | --- |
| Engine | PostgreSQL |
| Endpoint | `qibodb-postgres.c1aueiqms02c.ap-southeast-1.rds.amazonaws.com` |
| Port | 5432 |
| Database name | `qibodb` |
| Admin user | `qibodbadmin` |
| Admin password | `<edited_for_privacy — ask Ale` or or find it yourself by SSHing into the DB server at `54.169.91.191` and looking inside the `qibodb.service` file, where the full connection string (including the password) is set as the `QIBO_DB_URI` environ variable. |

Full connection string (used internally by the Flask app):

```java
postgresql+psycopg2://qibodbadmin:<password>@qibodb-postgres.c1aueiqms02c.ap-southeast-1.rds.amazonaws.com:5432/qibodb
```

> **Note:** The AWS Console login URL points to `ap-southeast-2`, but the RDS instance endpoint is in `ap-southeast-1`. This may be a console redirect quirk or a genuine mismatch. Someone with AWS Console access should verify which region the RDS instance actually lives in and update this page accordingly.

### AWS Console access

| Property | Value |
| --- | --- |
| Account ID / alias | `nus-cqt-nqch` |
| Username | `<edited_for_privacy — ask Ale>` |
| Password | `<edited_for_privacy — ask Ale>` |
| Console URL | <https://ap-southeast-2.signin.aws.amazon.com/oauth?client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&code_challenge=42eQv5OLAHunC-wXzAI2wTdjkdcpbK1OcdMhyrBWHBU&code_challenge_method=SHA-256&response_type=code&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fca-oauth-flow-id%3D31e0%26hashArgs%3D%2523%26isauthcode%3Dtrue%26oauthStart%3D1773808365811%26state%3DhashArgsFromTB_ap-southeast-2_15f3c13019dec53e> |

## Schema

The database has three tables. Relationships between them are by convention (matching `hash_id` / `run_id` values), not enforced via foreign keys.

### Table `calibration`

Stores calibration snapshots as ZIP archives. Deduplicated by `hash_id` — no two rows share the same hash.

| Column       | Type     | Notes                                           |
|:-------------|:---------|:------------------------------------------------|
| `id`         | INTEGER  | Primary key                                     |
| `hash_id`    | VARCHAR  | Indexed, unique in practice (deduplication key) |
| `notes`      | VARCHAR  | Nullable                                        |
| `created_at` | DATETIME | Server-set on insert                            |
| `filename`   | VARCHAR  | Original ZIP filename                           |
| `data`       | BLOB     | Full ZIP archive (binary)                       |

### Table `result`

Stores experiment results as ZIP archives, each linked to a calibration via `hash_id`.

| Column       | Type     | Notes                                                               |
|:-------------|:---------|:--------------------------------------------------------------------|
| `id`         | INTEGER  | Primary key                                                         |
| `hash_id`    | VARCHAR  | Indexed; corresponds to `calibration.hash_id`                       |
| `name`       | VARCHAR  | Indexed; logical grouping — in practice always `"experiment_group"` |
| `run_id`     | VARCHAR  | Nullable, indexed; 14-char timestamp e.g. `20260309202748`          |
| `notes`      | VARCHAR  | Nullable                                                            |
| `created_at` | DATETIME | Server-set on insert                                                |
| `filename`   | VARCHAR  | Original ZIP filename                                               |
| `data`       | BLOB     | Full ZIP archive (binary)                                           |

Deduplication rule: if a row with the same `(hash_id, name, run_id)` already exists, the insert is skipped and the API returns `"presaved"`.

### Table `best_run`

Tracks which run is considered the current best for a given calibration. Append-only — no uniqueness constraint. The "current best" is always the row with the highest `id` (most recently inserted). History is preserved.

| Column                | Type     | Notes                                                                           |
|:----------------------|:---------|:--------------------------------------------------------------------------------|
| `id`                  | INTEGER  | Primary key                                                                     |
| `calibration_hash_id` | VARCHAR  | Indexed; corresponds to `calibration.hash_id`                                   |
| `run_id`              | VARCHAR  | Indexed; together with `calibration_hash_id` identifies a specific `result` row |
| `created_at`          | DATETIME | Server-set on insert                                                            |

## Flask API layer (EC2 · 54.169.91.191)

The database is not accessed directly. All reads and writes go through an HTTP API built with Flask, served by Gunicorn.

| Property | Value |
| --- | --- |
| Source repo | `tanvirulz/nqch-QDB` (GitHub) |
| Deployed to | `/opt/qibodb` on EC2 `54.169.91.191` |
| Runs as | systemd service (`/etc/systemd/system/qibodb.service`, the config file that tells the OS how to run and manage this process) |
| Process manager | Gunicorn |
| Auth mechanism | `QIBO_API_TOKEN` (set as env var in the service file) |
| OS user with sudo | `ubuntu` |

### Why `/opt/qibodb`

`/opt/` is a standard Linux folder used for software that was installed manually (e.g. by cloning a repo) rather than through `apt` or another package manager. It's the conventional place to put standalone applications that you deploy yourself. The `nqch-QDB` repo was cloned here because it's a standalone application deployed outside of any package management flow.

### Why systemd

`qibodb.service` is a configuration file that tells the operating system how to run the Flask/Gunicorn process. Think of it as instructions for the OS: "run this program, keep it alive, and restart it if it crashes." Because it's registered with systemd (Linux's built-in service manager), the process starts automatically when the server boots and you can start, stop, or restart it using `systemctl` commands instead of manually running scripts. The service file was created and placed manually at `/etc/systemd/system/qibodb.service` during initial server setup.

### Managing the service

To inspect or modify the service (e.g. rotating the API token), you need to SSH into the DB server as `ubuntu`. Your SSH public key must have been added to this user's `authorized_keys` by someone with existing access (currently Ale).

```java
ssh -i ~/.ssh/id_ed25519_github1 ubuntu@ec2-54-169-91-191.ap-southeast-1.compute.amazonaws.com
```

The service file lives at:

```java
/etc/systemd/system/qibodb.service
```

To edit it:

```java
sudo nano /etc/systemd/system/qibodb.service
```

After any change to the service file, you must run:

```java
sudo systemctl daemon-reload
sudo systemctl restart qibodb
```

`daemon-reload` tells systemd to re-read all service files from disk — without this, systemd continues using the old cached version of `qibodb.service` and your changes have no effect. `restart qibodb` then stops the running Gunicorn process and starts a fresh one with the updated configuration (including any new environment variables like a rotated API token). You need to run both commands together any time you edit the service file.

### Verifying the service is running

```java
# Check service status
sudo systemctl status qibodb

# Find the Gunicorn PIDs
pgrep -f "gunicorn"

# Inspect the environment variables the process is running with
cat /proc/<PID>/environ | tr '\0' '\n' | grep QIBO

```

## API token rotation

The API token authenticates requests between the benchmarking client and the Flask server. It must match on both sides. If you only update one side, auth fails.

### Step 1: Update the server (DB EC2 · 54.169.91.191)

SSH in as `ubuntu`, edit the service file, replace the `QIBO_API_TOKEN` value, then reload and restart:

```java
sudo nano /etc/systemd/system/qibodb.service
# replace the QIBO_API_TOKEN value with the new token
sudo systemctl daemon-reload
sudo systemctl restart qibodb

```

### Step 2: Update the client (cqt-ale-server · 54.251.1.211)

SSH in as `benchmarking`, update `.env_user`, then restart the polling script:

```java
nano ~/.env_user
# replace CQT_API_TOKEN value with the same new token

```

Then restart `run_benchmarking.sh` — see the [2nd Part: AWS](../../Lab%20Projects/Benchmarking%20Pipeline/2nd%20Part_%20AWS.md) section in the Part 2: AWS page for how to stop and restart the screen session.
