
[Security](../Security.md)

# Environment variables

# File .env\_user

The file `.env_user` is a file stored outside of any repo, one level up, which means it is destined to live inside your `user` folder (if in the server) or project folder (if in your local machine). This is intentionally done this way as an extra safety to prevent anyone from accidentally pushing secrets to git repos when they forget to properly use `.gitignore`.

For now, the idea is this file contains all the environment variables needed to operate smoothly across repos, servers, and databases that are actively used by CQT-S14 people on a daily basis.

At the moment, there is no SRE person to oversee credentials, tokens, permissions etc so, at least, having a “single source of truth” for env vars per-user helps minimize damage spreading and makes it a bit easier to handle credential rotations when those are compromised.

**As of** 2026-04-05**,**

The below are the environment variables actively being used, thus in `.env_user` file:

```java
CQT_SERVER_URL=http://54.169.91.191 
CQT_API_TOKEN=<edited_to_keep_things_secure>
```

The `CQT_SERVER_URL` points to the HTTP Requests API exposed by Tanvirul’s DataBase client, ie. where one sends GET and POST requests directly to the DB. The `CQT_API_TOKEN` is the bearer API token for authenticating when sending such requests.

> [!IMPORTANT]
> Notice that in order to pass the variables in `.env_user` file to certain processes you may need to store the variables in the file preceded by “export” statement, like:
>
> ```java
> export CQT_SERVER_URL=http://54.169.91.191 
> export CQT_API_TOKEN=\<edited_to_keep_things_secure\>
> ```

## Legacy / dead references

The environment variables `CQT_QIBODB_HOST` and `CQT_QIBODB_KEY` are defunct. They may still appear in `/home/benchmarking/.env_user` on `cqt-ale-server` and/or in a `.env` file inside `/opt/qibodb` on the DB server, but they are not used by anything. Confirmed absent from all branches and commits of `CQT-reporting` via:

```java
git grep "QIBODB_KEY\|QIBODB_HOST\|QIBODB" $(git rev-list --all)
# returned nothing
```

These should be ignored if encountered in old documentation or scripts.

## Github PATs

### PAT for benchmarking user on cqt-ale-server

A PAT enables `benchmarking` user in `cqt-ale-server` to push to `gh-pages` branch on official [CQT-reporting](https://github.com/Scinawa/CQT-reporting) repo. The PAT gives read&write permission. This enables the part of the benchmarking pipeline in charge of publishing the benchmarking report to function.

1. **Create the PAT:** [Ale Luongo](https://github.com/Scinawa) needs to create the PAT as it pushes to his Github repo

2. **Store the PAT:**  It is currently stored in `.env_user` in variable `GITHUB_TOKEN` so that it can be used within the push command directly as in

```java
# Push using PAT TOKEN
git push https://$GITHUB_TOKEN@github.com/Scinawa/CQT-reporting.git gh-pages

# in .env_user
GITHUB_TOKEN=<edited_to_keep_things_secure>
```
