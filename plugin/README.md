# Stackwarden for Claude Code

Scans a repo for the services it runs on, such as hosting, databases, email, payments and AI APIs, and files them into the right project in your Stackwarden app. It also checks the repo for common security problems and gives you a brief for fixing them. It can tell you what a project runs on and where each key is configured.

Code and secret values are never sent to Stackwarden. The scan sends only service names, env var names, file paths and line numbers, and you confirm the findings before anything is sent. The security checklist goes with them, as check names, states, paths and line numbers, and only your space's members see it in Stackwarden.

## Install

The easiest way is to say "I want to use Stackwarden: https://stackwarden.vercel.app/agents.md" in a repo. Claude Code reads that guide and installs the plugin itself. It then asks you only to run `/reload-plugins` and to sign in through `/mcp`. In auto mode, Claude Code's safety check does not let it install a plugin, so it asks you to.

To install it yourself, run these in a terminal (tested with Claude Code 2.1.309):

```bash
claude plugin marketplace add https://github.com/cermakjan-hub/stackwarden-plugin.git
claude plugin install stackwarden@stackwarden
```

In the VS Code extension, open this link instead, for example from your browser's address bar. It opens Manage plugins on Stackwarden, and nothing installs until you choose where:

```text
vscode://anthropic.claude-code/install-plugin?plugin=stackwarden&marketplace=https%3A%2F%2Fgithub.com%2Fcermakjan-hub%2Fstackwarden-plugin.git
```

Then run `/reload-plugins`, then `/mcp`, choose `plugin:stackwarden:stackwarden` and sign in. Your browser opens Stackwarden, where you allow Claude Code; there is no key to copy. Claude Code keeps the sign-in fresh, and you can revoke it under Settings, Connected apps.

The plugin connects to https://stackwarden.vercel.app. To use another Stackwarden, such as a local one, set `STACKWARDEN_URL` before starting Claude Code, for example `STACKWARDEN_URL=http://localhost:3000`.

## Updates

Claude Code leaves auto-update off for a marketplace outside Anthropic's, so new versions don't arrive by themselves. To have them install without asking, in a terminal run `claude`, then `/plugin`, Marketplaces, stackwarden, Enable auto-update. Then you'll see `Plugin updated: stackwarden · Run /reload-plugins to apply` when there's a new one.

Without it, a scan says when a newer version is out, and this gets it, followed by `/reload-plugins`:

```bash
claude plugin update stackwarden@stackwarden
```

## Use

- In a repo, say "I want to use Stackwarden", or run `/stackwarden:start`. Claude Code scans the repo, shows what it found, and files it once you confirm.
- `/stackwarden:scan` scans the current repo again. Each new finding waits on the project's page in the app for you to approve or dismiss, with the Vercel project it is picked where Stackwarden can tell.
- Ask "what does this project run on?" or "where is `STRIPE_SECRET_KEY` set?" to look it up in Stackwarden.

The security checks only report. They say what's wrong in plain words and give a brief for each fix, you have it fixed however you work, and scanning again says what changed. Nothing stops you from sending. A passing result means no rule matched in what was scanned, not that the app is safe. If a finding is handled, for example because you changed a leaked key at its provider, list it in `.stackwarden/exceptions.json`. It stays counted, so an exception never hides anything.

Mistakes in an app's logic, such as who can see whose data, need an AI review. For that, install Anthropic's Claude Security plugin (`/plugin install claude-security@claude-plugins-official`) and run `/claude-security`. The next scan shows what it found, labelled as AI and apart from the checklist, and sends none of it.

Claude also reads the places no rule can judge, with their values masked. What it thinks may be a real key is labelled as coming from an AI. It can be wrong, it isn't a full list, and it doesn't count toward the result.

The scan needs Node.js 22 or newer and git for its full result. While it runs, a hook blocks Claude Code from reading dotenv files, keys and other secret files, and lets the shell run only the scanner until Claude looks up the project. For the rest of the session, it also blocks searches that would print lines holding keys. The hook refuses every tool call it guards if Node.js cannot start.
