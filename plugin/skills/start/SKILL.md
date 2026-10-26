---
name: start
description: Sets this repo up in Stackwarden in one go. It checks that Claude Code is signed in to Stackwarden, scans the repo for the services it runs on, files them after the user confirms once, and says what to connect next. Use when the user wants to start using Stackwarden in this repo, or to set it up.
when_to_use: 'For example "I want to use Stackwarden", "use Stackwarden here", "set up Stackwarden", "get started with Stackwarden", "connect this repo to Stackwarden" or "add this project to Stackwarden".'
allowed-tools: mcp__plugin_stackwarden_stackwarden__get_stack
---

# Start using Stackwarden in this repo

The user wants this repo in Stackwarden with as little effort as possible. Do the work yourself. Ask only for what needs them: signing in once, and one confirmation before anything is sent.

## 1. Check that Claude Code is signed in

Stackwarden's tools are named `mcp__plugin_stackwarden_stackwarden__*`. If they are missing, or a call to one fails with 401, says sign-in is needed, or cannot connect, tell the user this, and stop until they are back:

> To connect Claude Code to Stackwarden, run `/mcp`, choose `plugin:stackwarden:stackwarden` and sign in. Your browser opens Stackwarden, where you allow Claude Code. Then tell me to continue.

When they are back, start again from this step. Only if sign-in reaches the wrong site, or they say their Stackwarden runs elsewhere (for example on `localhost`), add that `STACKWARDEN_URL` must name it before Claude Code starts.

## 2. Scan the repo and file it

Use the Skill tool to run `stackwarden:scan`, and follow it to the end. It finds the services on this machine, shows them with the project they go to, and asks the user once before it sends anything. If they cancel, stop here.

## 3. Say what comes next

Call `get_stack` with the project ID the scan reported. Then tell the user, in a few short lines:

- The project's link, and how many services wait for their approval, which they approve or dismiss on the project's page.
- For each provider in `toConnect`: connecting it imports its projects and usage, with the link to connect it. They create the token at the provider and paste it into Stackwarden. Never ask for a token or key here.
- From now on, in this repo, Claude Code says at the start of a session when the code has changed since the scan, and records the services it sets up, so they wait for approval on the project's page.
