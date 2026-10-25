---
name: stack
description: Answers what this project runs on (its hosting, databases, email, payments, AI and other services) and where each key is configured, from Stackwarden, and says what Stackwarden does not know yet. Use when the user asks what the project uses, which services or providers it depends on, or where an env var or key lives.
allowed-tools: Bash(node "${CLAUDE_SKILL_DIR}/../scan/scripts/scan.mjs" *) mcp__plugin_stackwarden_stackwarden__match_repo mcp__plugin_stackwarden_stackwarden__get_stack
---

# What this project uses

1. Scan the repo on this machine. It reads no secret files and sends nothing:

   ```bash
   node "${CLAUDE_SKILL_DIR}/../scan/scripts/scan.mjs" .
   ```

   Keep `repo`, `linkedIds.vercelProjectId`, each finding's `service` and `name`, and the env var names in `envVars`. Nothing else from the report is needed.

2. Call `match_repo` with `remote_url` set to `repo.remote`, `vercel_project_id` if present, and `name` set to `repo.name`.

   - A match: continue with its project ID.
   - Candidates: ask the user which project it is.
   - No match: tell the user the repo is not in Stackwarden yet and suggest saying "I want to use Stackwarden". Stop.

3. Call `get_stack` with the project ID.

4. Compare what the code shows with what Stackwarden knows:

   - Findings whose `service` is not the provider of any service in `get_stack`, which shows each provider in brackets, as in `harbor-crm [vercel]`: the code uses them, and Stackwarden does not list them yet.
   - Services marked "not seen in the latest scan".
   - The last scan's commit, when it is not `repo.commit`: Stackwarden's view is from another commit.
   - Accounts not read for more than a day, or needing attention, and providers in `toConnect`.

5. Answer the user's question briefly from `get_stack`: the services by kind, and for keys, the env var name and where its value lives (never a value). Then say in one or two lines what Stackwarden may not know yet, from step 4, when it bears on the question. Offer `/stackwarden:scan` when the code has services Stackwarden lacks. Give the project link.
