# Stackwarden for Claude Code

Scans a repo for the services it runs on, such as hosting, databases, email, payments and AI APIs, and files them into the right project in your Stackwarden app. It can also tell you what a project runs on and where each key is configured.

Code and secret values never leave your machine. The scan sends only service names, env var names, file paths and line numbers, and you confirm the findings before anything is sent.

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

## Use

- In a repo, say "I want to use Stackwarden", or run `/stackwarden:start`. Claude Code scans the repo, shows what it found, and files it once you confirm.
- `/stackwarden:scan` scans the current repo again. Each new finding waits on the project's page in the app for you to approve or dismiss, with the Vercel project it is picked where Stackwarden can tell.
- Ask "what does this project run on?" or "where is `STRIPE_SECRET_KEY` set?" to look it up in Stackwarden.

The scan needs Node.js 22 or newer. While it runs, a hook blocks Claude Code from reading dotenv files, keys and other secret files, and lets the shell run only the scanner until Claude looks up the project. The hook refuses every tool call it guards if Node.js cannot start.
