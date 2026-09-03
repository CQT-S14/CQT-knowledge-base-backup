# CQT-S14 Confluence Backup

This repository is an automated, version-controlled mirror of the CQT-S14 lab's
Confluence space, converted to Markdown. It exists as a disaster-recovery
safety net: Confluence Cloud's Free tier can deactivate a site after a period
of inactivity, and this repo is what survives even if the Confluence account
itself doesn't.

**Direction of travel: Confluence → this repo, one-way.** This is a backup
archive, not a two-way sync. It does not push changes back into Confluence.

## What's in this repo

```
/content/                    the Markdown mirror of the Confluence space,
                              folder structure matches the Confluence page tree
/.github/workflows/          scheduled export + commit job (pending)
/README.md                   this file
```

## Tooling

Content is exported using
[`confluence-markdown-exporter`](https://github.com/Spenhouet/confluence-markdown-exporter)
(CLI command: `cme`) — MIT-licensed, actively maintained, and built for
exactly this job. It preserves headings, tables, code blocks, images, and
attachments (including PDFs), and only re-exports pages that changed since
the last run.

## Local installation

Tested working on macOS (Apple Silicon), September 2026:

```bash
curl -LsSf uvx.sh/confluence-markdown-exporter/install.sh | sh
```

This installs two equivalent commands, `cme` and `confluence-markdown-exporter`,
available globally on the machine (not scoped to this repo — same as any
other CLI tool like `git`). Confirm it installed with:

```bash
cme --help
```

## Configuration

Credentials and settings are stored in a single config file, managed via:

```bash
cme config edit auth.confluence      # set Confluence URL, email, API token
cme config set export.output_path=./content
```

Run `cme config path` at any time to see exactly where that config file lives
on your machine.

## Credentials

The config file lives at an OS-standard user directory entirely outside this repo — on
macOS, `~/Library/Application Support/confluence-markdown-exporter/app_data.json`
(Linux/Windows use their own equivalent standard locations).

The automated GitHub Actions workflow uses a separate mechanism entirely:
Confluence credentials are stored as **encrypted GitHub repository secrets**
(Settings → Secrets and variables → Actions) and injected as environment
variables at runtime, inside a throwaway container. They're never written to
a file in this repo either.

Net result: there is no credential-shaped file that could ever land in this
working tree.

## Restoring from this backup

Because this is a one-way mirror, "restoring" means reading the content
here, not pushing it back into Confluence automatically:
- Browse it directly on GitHub — Markdown renders natively in the repo UI.
- Or clone the repo locally for offline access to everything.

## What's not captured

To be confirmed as testing continues — some newer Confluence content types
(e.g. whiteboards, databases) may not be supported by the exporter. Check
the [exporter's current feature list](https://github.com/Spenhouet/confluence-markdown-exporter)
and update this section once verified against this space's actual content.