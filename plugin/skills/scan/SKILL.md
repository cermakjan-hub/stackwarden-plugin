---
name: scan
description: Scans this repo for the services it runs on (hosting, databases, email, payments, AI and more) and files them into the right Stackwarden project. Use when the user runs /stackwarden:scan or asks to scan or rescan this repo into Stackwarden, and when the start skill hands over to it. Code and secret values never leave the machine; only names, file paths and line numbers are sent.
allowed-tools: Bash(node "${CLAUDE_SKILL_DIR}/scripts/scan.mjs" *) Read Grep Glob mcp__plugin_stackwarden_stackwarden__match_repo
hooks:
  PreToolUse:
    - matcher: "Read|Grep|Bash|PowerShell|mcp__plugin_stackwarden_stackwarden__match_repo|mcp__plugin_stackwarden_stackwarden__submit_scan"
      hooks:
        - type: command
          command: node "${CLAUDE_PLUGIN_ROOT}/skills/scan/scripts/guard.mjs"
          onFailure: block
---

# Scan this repo into Stackwarden

You find the services this repo uses and file them into Stackwarden. The repo's code and any secret values stay on this machine: Stackwarden receives service names, kinds, env var **names**, file paths with line numbers, and the git remote. Never code, file contents or values.

## Rules that always apply

- Never open `.env`, `.env.local`, `.envrc` or any other `.env*` or `*.env` file except `.env.example`, `.env.sample` and `.env.template`. Never open `*.pem`, `*.key`, `id_rsa*`, `.git/config`, or anything in `secrets/` or `.ssh/`. A hook blocks these; do not try to work around it.
- Until you call `match_repo`, the shell may run only the scanner command in step 1, and Read and Grep stay inside this repo. Use Read, Grep and Glob to look at code. Do not run `git` or other commands: the report already has the repo's remote, branch and commit.
- When you search with Grep, pass a `glob` that ends in a file extension (for example `*.ts`, `*.{ts,tsx}` or `src/**/*.py`) or a `type` such as `ts` or `py`. Globs such as `*`, `.e*` or `*.json` are refused, because they could reach secret files.
- Never put a value in what you send: no keys, tokens, passwords, connection strings or code. Env var names only (for example `STRIPE_SECRET_KEY`), each free-text field under 200 characters. Stackwarden rejects the whole call if anything looks like a secret.
- If the scanner fails to start (for example, `node` is not installed), stop and tell the user the scan needs Node.js 22 or newer. Do not scan by reading files yourself.
- If the user cancels at the confirmation step, stop. Send nothing.

## Steps

### 1. Run the scanner

```bash
node "${CLAUDE_SKILL_DIR}/scripts/scan.mjs" .
```

It prints a JSON report and sends nothing anywhere. The report has:

- `repo`: `name`, `remote` (already cleaned of credentials), `branch`, `commit`.
- `linkedIds`: `vercelProjectId` when the repo is linked to a Vercel project.
- `findings`: one per service, with `service` (a slug), `name`, `kind`, `strength`, `source: "scanner"`, `evidence` (signal, file, line, detail) and `envVars` (each name with the files and lines that use it).
- `envVars`: every env var name the scanner saw, including ones no rule assigned to a service.
- `skippedSecretFiles`: how many secret files it saw and left unread.
- `withheldItems`: how many file paths or details it left out because they looked like secret values, such as a file named after a token. If it is above 0, tell the user in the summary; never try to find or add them yourself.

The same repo always gives the same report.

### 2. Review the code for what the rules miss

The scanner reads dependencies, config files, env var names, containers, CI workflows and Terraform. Look briefly for services it cannot see:

- APIs called over plain HTTP (`fetch`, `axios`, `requests`) to a third-party host.
- Incoming webhooks (routes named like `webhook`, `callback`, `hooks`).
- Scheduled jobs (`cron`, `schedule`, job runners) and what they call.
- Env var names from the report's `envVars` that no finding owns, which often belong to a service.

Keep it short: a few targeted Grep calls with a `glob`, and Read only the files those point to. Each extra finding needs a real file and line. Add it with `source: "ai"`, `strength: "weak"`, and evidence `{"signal": "ai_review", "path": "<file>", "line": <n>, "detail": "<what you saw, under 200 characters, no values>"}`. Use a lowercase slug for `service` (see `detection.md` for known slugs and kinds; read it only if you need it). Findings you make always wait for the user's approval on the project page.

### 3. Find the project

Call `match_repo` with `remote_url` set to `repo.remote` (or `null`), plus `vercel_project_id` from `linkedIds` if present, and `name` set to `repo.name`.

- A match: file into that project ID.
- Candidates: ask the user which project it is, or whether to use a new one.
- New: offer a project named after the repo. The user may rename it.

### 4. Confirm with the user

Show a short summary and ask the user to confirm or edit before anything is sent:

```text
Stackwarden scan of <repo name> → <project name> (new | existing)

  Supabase       database   strong   supabase/config.toml, package.json:5
                 keys: SUPABASE_SERVICE_ROLE_KEY (src/lib/db.ts:3)
  Stripe         payments   strong   package.json:8, STRIPE_SECRET_KEY
  Sentry         monitoring medium   package.json:9
  Acme webhooks  other      found by Claude  src/hooks.ts:12

Each one waits for your approval on the project page, with the Vercel project it is picked where Stackwarden can tell.
Send this to Stackwarden? (yes / edit / cancel)
```

Let the user drop findings or change the project. If you scanned this repo before in this session, say what changed.

### 5. File the scan

Call `submit_scan` once, with:

- `project`: the project ID from `match_repo`, or the name for a new project.
- `repo`: the report's `repo` object as it is.
- `findings`: only the confirmed findings, each exactly as in the report (or as you built it in step 2).
- `scanner_version`: the report's `scannerVersion`.

### 6. Report what happened

Print what Stackwarden says it did: what was updated, what waits for the user to approve, anything no longer seen, and the link to the project, where the user approves or dismisses each one.

## When something goes wrong

- **Stackwarden's tools are missing, or a call fails with 401 or a connection error:** the plugin is not signed in, or its access was revoked. Ask the user to run `/mcp` in this session, choose `plugin:stackwarden:stackwarden` and sign in, then tell you to continue. Only if sign-in reaches the wrong site, or they say their Stackwarden runs elsewhere (for example on `localhost`), add that `STACKWARDEN_URL` must name it before Claude Code starts.
- **"looks like a secret value" or "Rejected: … look like secret values":** a field held something shaped like a secret. Remove or rewrite that field (names only) and submit again.
- **"This commit was already filed":** nothing changed since the last scan of this commit. Tell the user; nothing else to do.
