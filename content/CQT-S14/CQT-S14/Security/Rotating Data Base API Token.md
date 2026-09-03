
[Security](../Security.md)

# Rotating Data Base API Token

- [Context: The underlying Database](#rotatingdatabaseapitoken-contexttheunderlyingdatabase)
- [The API Token to Rotate](#rotatingdatabaseapitoken-theapitokentorotate)
  - [Step 1. On ubuntu@54.169.91.191, update the service file and restart](#rotatingdatabaseapitoken-step1onubuntu5416991191updatetheservicefileandrestart)
  - [Step 2. On benchmarking@54.251.1.211, update .env\_user and restart the script](#rotatingdatabaseapitoken-step2onbenchmarking542511211updateenv-userandrestartthescript)

# Context: The underlying Database

The database was built and set up by Tanvirul, and runs on a dedicated EC2 instance (`54.169.91.191`) separate from the benchmarking EC2 (`54.251.1.211`) where the `benchmarking` user lives. On that dedicated EC2, gunicorn spins up the qibodb Flask server (cloned from [tanvirulz/nqch-QDB](https://github.com/tanvirulz/nqch-QDB) into `/opt/qibodb/`) whose sole purpose is to expose an authenticated HTTP API that allows clients to upload, download, and query experimental results and calibrations.

The Flask server does not store data itself — it is an HTTP API layer that sits in front of a PostgreSQL database hosted on AWS RDS in the `ap-southeast-1` region (Singapore). The full connection URI, as found in `qibodb.service` and confirmed from the running process, is:

```java
postgresql+psycopg2://qibodbadmin:{password_edited}@qibodb-postgres.c1aueiqms02c.ap-southeast-1.rds.amazonaws.com:5432/qibodb
```

Breaking that down:

- **engine**: `postgresql+psycopg2` — Postgres, using the psycopg2 Python driver
- **user**: `qibodbadmin` — the Postgres user the Flask server authenticates as
- **password**: `{password_edited}` — the password for that Postgres user, which should also be rotated (via the AWS RDS console, not from the EC2)
- **host**: `qibodb-postgres.c1aueiqms02c.ap-southeast-1.rds.amazonaws.com` — the AWS-managed RDS instance, not an EC2 machine
- **port**: `5432` — standard Postgres port
- **database name**: `qibodb`

This password is separate from `QIBO_API_TOKEN` — it is not something clients ever see or use. It is only used internally by the Flask server to connect to RDS. To rotate it you would need access to the AWS console for this account.

# The API Token to Rotate

The below are two different names for the same single Data Base auth API Token.

- `CQT_API_TOKEN` is the name of the token in file `.env_user` used by `CQT-experiments` and `CQT-reporting` repos.
- `QIBO_API_TOKEN` is the name of the token in `qibodb.service`, the systemd file that defines how the qibodb Flask server is configured and launched — including which token it expects incoming requests to carry for authentication.

## Step 1. On `ubuntu@54.169.91.191`, update the service file and restart

First, connect to the server. This requires your SSH public key to have been added to the `ubuntu` user's `~/.ssh/authorized_keys` by Ale beforehand:

```java
ssh ubuntu@54.169.91.191
```

The `ubuntu` user has been granted `sudo` permissions on this machine. Any time you need to rotate credentials or modify service config, you must be on a `sudo`-enabled user like `ubuntu`.

Some context on where things live:

- `/opt/` is the standard Linux directory for third-party software installed manually — not part of the OS. The qibodb Flask server lives at `/opt/qibodb/` because it was cloned there from [tanvirulz/nqch-QDB](https://github.com/tanvirulz/nqch-QDB) and set up manually.
- `/etc/systemd/system/` is where Linux's service manager (systemd) looks for service definitions. `qibodb.service` lives there so that systemd knows how to start the gunicorn/Flask server automatically on boot, restart it if it crashes, and run it with the correct environment variables (including `QIBO_API_TOKEN`).

Generate a new token:

```java
# This uses Python's cryptographically secure random generator to produce a
# 32-byte random string encoded as URL-safe base64. It is a valid API token
# for this DB because the qibodb server does not generate or store tokens
# itself — it simply accepts whatever string is set as QIBO_API_TOKEN and
# checks incoming requests against it. Any sufficiently random string works.

python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

Copy the output. Then edit the service file:

```java
sudo nano /etc/systemd/system/qibodb.service
# replace QIBO_API_TOKEN=<old_token> with QIBO_API_TOKEN=<new_token>
# save with Ctrl+O, exit with Ctrl+X
```

Reload and restart:

```java
# daemon-reload tells systemd to re-read the qibodb.service file you just edited
# restart qibodb stops and restarts the gunicorn process with the new environment variables
sudo systemctl daemon-reload
sudo systemctl restart qibodb
sudo systemctl status qibodb  # verify it says "active (running)"
```

## Step 2. On `benchmarking@54.251.1.211`, update `.env_user` and restart the script

Connect to the benchmarking server. Again, your SSH public key must have been added to the `benchmarking` user's `~/.ssh/authorized_keys` by Ale:

```java
ssh benchmarking@54.251.1.211  # this is the right public ip
```

Update the token:

```java
nano ~/.env_user
# replace CQT_API_TOKEN=<old_token> with the new token from step 1
# save with Ctrl+O, exit with Ctrl+X
```

`run_benchmarking.sh` runs inside a `screen` session so it keeps running after you disconnect. To restart it:

```java
# List running screen sessions to find the one running run_benchmarking.sh
screen -ls

# Reattach to it (replace <session_id> with the one from screen -ls)
screen -r <session_id>

# Once inside, kill the running script
Ctrl+C

# Restart it
./run_benchmarking.sh

# Detach from screen WITHOUT killing the script
Ctrl+A then D
```

The script will now keep running in the background even after you close your SSH session.

If you only do Step 1, the server accepts the new token but the benchmarking client still sends the old one → auth fails. If you only do Step 2, the opposite. Both sides must match.
