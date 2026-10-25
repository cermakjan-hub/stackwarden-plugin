# Stackwarden plugin for Claude Code

The Claude Code plugin for [Stackwarden](https://stackwarden.vercel.app), which keeps track of what each of your projects runs on, and where each key is configured, without holding a key's value. What the plugin does, how to install it and what it sends: [plugin/README.md](plugin/README.md).

Stackwarden is in early access: signing in works for invited accounts only.

## Install

In Claude Code, in any repo, say "I want to use Stackwarden: https://stackwarden.vercel.app/agents.md". Or install it yourself:

```bash
claude plugin marketplace add https://github.com/cermakjan-hub/stackwarden-plugin.git
claude plugin install stackwarden@stackwarden
```

## This repo

Each plugin release is published here from Stackwarden's private repository, so a change made here directly is overwritten by the next one. Report problems and ideas as issues here.

## License

Apache License 2.0; see [LICENSE](LICENSE). The scanner bundles third-party packages, listed with their licenses in [plugin/THIRD_PARTY_NOTICES.md](plugin/THIRD_PARTY_NOTICES.md).
