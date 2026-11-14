# Paperclip starter kit

A setup guide and two shared skills for running a small coding team of agents in [Paperclip](https://paperclip.ing): a lead, a developer, a tester and a reviewer.

Everything here was learned the hard way on a real project. Each fix is tied to a problem that actually stalled work, wasted tokens or buried the owner in comments.

Tested on Paperclip **2026.1023.0**, in `local_trusted` mode with Claude Code agents. Paperclip changes quickly, so check anything version-sensitive before relying on it.

## Using this with an LLM

Give your LLM (Claude Code, Cowork, or anything with access to your Paperclip instance) this prompt:

> Read `SETUP.md` in https://github.com/camball96/paperclip-starter-kit. Then audit my Paperclip instance against it and apply the setup checklist. Ask me before changing anything you're unsure about, and tell me what you changed.

It needs to be able to reach your Paperclip API (by default `http://localhost:3100`) and the repo you want the agents to work on.

## What's here

| Path | What it is |
|---|---|
| [`SETUP.md`](SETUP.md) | Every problem we hit, its cause, the fix and how to check it. Then a setup checklist and the API calls to apply it. |
| [`skills/paperclip-team-workflow/`](skills/paperclip-team-workflow/SKILL.md) | How the lead turns a ticket into issues and moves it to a pull request without stalling |
| [`skills/paperclip-team-reporting/`](skills/paperclip-team-reporting/SKILL.md) | How every agent writes comments and handovers, so the owner reads three lines, not thirty |
| [`templates/agents/`](templates/agents/) | Instruction templates for the Lead, Dev, QA and Review agents |

## Installing the skills

In Paperclip, go to **Skills → New → Import from GitHub** and import this repo. Then attach:

| Agent | Skills |
|---|---|
| Lead | `paperclip-team-workflow`, `paperclip-team-reporting` |
| Dev, QA, Review | `paperclip-team-reporting` |

Project-specific skills (how to run your tests, how to open a PR in your repo, your QA procedure) belong in the project's own repo, next to the code they describe. See [Project skills](SETUP.md#project-skills) in `SETUP.md`.
