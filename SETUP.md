# Setting up a Paperclip coding team

This guide is written for an LLM configuring someone's Paperclip instance, and for the person reading along. Part 1 lists every problem we hit, as symptom, cause, fix and check. Part 2 is the setup checklist. Part 3 is the API calls that apply it.

Tested on Paperclip **2026.1023.0**, `local_trusted` mode, Claude Code agents (`claude_local` adapter). The API in Part 3 is undocumented and was observed on that version. Confirm each call against the instance before relying on it.

**If you are an LLM applying this:** audit first and report what differs from Part 2. Ask the owner before changing agent models, permissions or anything in their app repo. Never delete issues, agents or skills without asking.

---

## Part 1: What goes wrong, and the fix

### Work stalls

#### 1. An agent never wakes up after its sub-tasks finish
- **Symptom:** sub-tasks are done, the parent issue sits untouched for hours. Runs list shows *"Cancelled because issue dependencies are still blocked"* or *"Scheduled retry suppressed because issue dependencies are still blocked"*.
- **Cause:** the lead marked its own issue as blocked by its own children (or blocked the build issue on its test and review issues). Paperclip will not wake the assignee of a blocked issue. When a handover lands on that issue, nothing runs.
- **Fix:** never use blockers inside a ticket. Hand work over by **assigning** the next issue. See `paperclip-team-workflow`.
- **Check:** no issue in an active ticket has `blockedBy` entries pointing at its own siblings or children.

#### 2. "Recovery needed" banner, and messages that never send
- **Symptom:** an issue shows *"Recovery needed. Execution was cancelled; its source was not recorded… N saved messages are waiting for recovery"*. Comments you post are queued and never start a run. "Send queued messages now" delivers them but still starts nothing.
- **Cause:** a run on that issue was cancelled without a verified stop record, so Paperclip refuses to start new runs there.
- **Fix:** message the same agent on a **different** issue (for example the parent) and ask it to carry on. That starts a run. Or open **Inspect run** on the stuck run.
- **Check:** the agent has a new run within a minute of your message.

#### 3. Runs fail on a usage limit and never resume
- **Symptom:** runs fail with *"You've hit your 5-hour limit · resets …"* or a Codex *"usage limit … try again at <date>"*. Nothing runs after the reset.
- **Cause:** Paperclip doesn't reschedule work when a provider limit resets.
- **Fix:** after the reset, message the lead on the parent issue. Reduce token use (see 13 and 14). Don't put an agent on a provider account that's already exhausted. Check the date in the error before assigning it work.
- **Check:** the run list shows a successful run after the reset time.

#### 4. An agent fails with "does not support this model"
- **Symptom:** every run fails instantly: *"ACP agent reported a terminal service failure. API Error … Claude Code 2.x does not support this model"*. The agent shows status `error`.
- **Cause:** the agent runs on the default **ACP** execution engine, whose bundled Claude Code is older than the model it was given.
- **Fix:** set the agent's execution engine to **Claude CLI** (`adapterConfig.engine: "cli"`), or pick a model the bundled version supports.
- **Check:** **Test your agent** on the agent's runtime page succeeds.

#### 5. A follow-up issue waits for the agent to be "woken"
- **Symptom:** an agent asked a question in a comment, then the issue sat idle after the owner replied.
- **Cause:** a plain comment with a question gives the agent nothing to wait on.
- **Fix:** questions for the owner go in a **question card** (an interaction on the issue). Answering it wakes the agent.

#### 6. The developer stops because the base branch moved
- **Symptom:** the dev agent hands back *"branch check failed, `origin/main` has moved"* and waits for a decision.
- **Cause:** its instructions said to stop when the branch isn't up to date.
- **Fix:** dev merges the base branch in (never rebases) and carries on. It stops only on a conflict.

### Results that can't be trusted

#### 7. Two test runs wipe each other
- **Symptom:** tests that pass alone fail in the full run. Seeded rows are missing before their spec runs. Sign-ins fail for valid accounts.
- **Causes:** (a) each agent defaults to **20 concurrent runs**; (b) the reviewer also ran the end-to-end suite; (c) every worktree shares one test database that each run resets.
- **Fix:** set `maxConcurrentRuns: 1` on every agent. Only QA runs e2e. Review runs type-check and unit tests and trusts QA's run on the same commit. Before running e2e, QA checks no other `playwright test` process is running.
- **Check:** the run list never shows QA and Review e2e runs overlapping.

#### 8. The reviewer approves a fix that doesn't exist
- **Symptom:** "Approve, the e2e blocker is resolved", but the branch hasn't changed since the failing run.
- **Cause:** the reviewer read the thread, not the branch, and approved before QA had posted.
- **Fix:** a result only counts for the commit it ran on. Review starts by comparing `HEAD` with the commit in QA's latest passing report. If they differ or QA hasn't reported, it stops.
- **Check:** every approval names a commit, and that commit matches QA's.

#### 9. A sign-in rate limit looks like wrong credentials
- **Symptom:** e2e sign-ins fail with "incorrect password" only in the full run.
- **Cause:** the app's auth library rate-limits sign-ins per IP (some default to as few as 3 attempts per 10 seconds). Every Playwright context looks like one IP.
- **Fix:** specs that sign in repeatedly set a distinct `x-forwarded-for` header per browser context.

### Environment

#### 10. The app's test server collides with Paperclip on port 3100
- **Symptom:** e2e runs hit Paperclip's UI, or can't bind their port.
- **Cause:** Paperclip listens on **3100**. Many Playwright configs use 3100 too, and with `reuseExistingServer` Playwright will happily test against whatever is already there.
- **Fix:** move the app's e2e port (for example to 3200) and make it overridable with an env var.

#### 11. Docker, Playwright or tsx can't find their files in agent runs
- **Symptom:** `docker compose` can't find its config. Playwright can't find a browser. `tsx` fails with a socket or path-length error.
- **Cause:** agent runs use a different `HOME` and a long temp path.
- **Fix:** set agent environment variables: `DOCKER_CONFIG=<owner home>/.docker`, `PLAYWRIGHT_BROWSERS_PATH=<owner home>/Library/Caches/ms-playwright` (macOS), `TMPDIR=/tmp`.

#### 12. Test containers clash between worktrees
- **Symptom:** `docker compose up` fails: *container name "…" is already in use*.
- **Cause:** Compose names the project after the folder, so each worktree is a different project fighting over the same fixed `container_name`.
- **Fix:** add a top-level `name: <project>` to `docker-compose.yml`. Clear the stale container once with `docker rm -f <name>`.

### Noise and cost

#### 13. Walls of text the owner can't act on
- **Symptom:** paragraphs on every issue, "nothing needed from you" posts, tables of issues created.
- **Causes:** (a) Paperclip posts the agent's text as comments, including progress notes between tool calls, not only its final message; (b) handover detail lands on the issue the owner watches; (c) instructions asked for a "summary" with five required parts.
- **Fix:** the `paperclip-team-reporting` skill. Agents write nothing between tool calls. The lead posts on the parent only as `Status / Needs you / Next`. Detail goes on child issues.

#### 14. Huge token use per run
- **Symptom:** the lead's runs use 0.5–2.5M tokens each and hit the 5-hour limit within an evening.
- **Cause:** instructions told it to read all the repo docs on every wake, and it re-read whole threads.
- **Fix:** on each wake, read only what woke it. Read the repo docs when planning a ticket.

#### 15. Issues nested four levels deep
- **Symptom:** you can't tell what an issue is for. Every fix round makes new issues.
- **Fix:** one parent per ticket, with exactly three children (Build, Test, Review). Titles start with the ticket key. Fix rounds reopen the same three. See `paperclip-team-workflow`.

#### 16. Issue prefix is meaningless
- **Cause:** the issue prefix (for example `ACME`) is derived from the company name when it's created. There's no setting for it in the UI.
- **Fix:** name the company after the product before creating issues. On an existing instance, put the ticket key at the start of every issue title.

### Git and pull requests

#### 17. Stacked PRs land in the wrong branch
- **Symptom:** a merged PR's changes never reach the main branch.
- **Cause:** the PR was based on a parent task branch. The parent merged first, and the child PR merged into the dead branch.
- **Fix:** one ticket, one PR, always based on the main branch. If work depends on unmerged work, it waits.

#### 18. Review looks at an empty branch
- **Cause:** test and review issues started in a fresh workspace.
- **Fix:** pin Test and Review to Build's execution workspace, so all three share one worktree and branch.

### Harmless

- *"Cancelled before issue reassignment"* and *"Cancelled because issue assignee changed"* runs are normal churn when issues are reassigned. Ignore them.

---

## Part 2: Setup checklist

Work through in order. Each item says how to check it.

### Instance

- [ ] **Company name** matches the product, so issue prefixes are readable (16). *Check:* new issues get a sensible prefix.
- [ ] **Paperclip version** recorded. *Check:* `GET /api/health` → `version`.

### Agents

Four agents. Names describe the job, so comments read clearly.

| Agent | Role | Suggested model | Template |
|---|---|---|---|
| Lead | Plans and routes; never writes code | Strongest available | [`templates/agents/lead.md`](templates/agents/lead.md) |
| Dev | Builds and commits | Mid-tier coding model | [`templates/agents/dev.md`](templates/agents/dev.md) |
| QA | Writes and runs e2e tests | Mid-tier coding model | [`templates/agents/qa.md`](templates/agents/qa.md) |
| Review | Reviews against the repo's rules | Strongest available | [`templates/agents/review.md`](templates/agents/review.md) |

For each agent:
- [ ] **Instructions** from the template, with `{{PLACEHOLDERS}}` filled in. *Check:* the agent's Instructions page shows them.
- [ ] **Execution engine** is Claude CLI when using a recent model (4). *Check:* Test your agent passes.
- [ ] **`maxConcurrentRuns: 1`** (7). *Check:* `runtimeConfig.heartbeat.maxConcurrentRuns`.
- [ ] **Environment** for Dev, QA and Review: `DOCKER_CONFIG`, `PLAYWRIGHT_BROWSERS_PATH`, `TMPDIR` (11), only if the project uses them.
- [ ] **AI account** has headroom. Check the provider isn't already rate-limited (3).
- [ ] **Skills** attached as in the README.

### The project repo

- [ ] **e2e port** is not 3100 and can be overridden (10).
- [ ] **`docker-compose.yml`** has a top-level `name:` (12).
- [ ] **Agent guidance** (`AGENTS.md` or similar) lists which commands agents may run and which are owner-only, and has hard limits: never merge, never push to the main branch, never run migrations.
- [ ] **Project skills** exist for running the tests and opening a PR (below).

### Project skills

Keep these in the project repo (for example `skills/`), because they go stale when the code changes:

| Skill | Covers |
|---|---|
| `<project>-qa` | The self-check a developer runs before handing over: gates, reading their own diff, sweeping for the class of change |
| `<project>-e2e` | Preflight (no other run, port free), the one command, and a table of failure symptoms and their usual causes |
| `<project>-pr` | Base branch, pre-push checks, and a PR description template under 30 lines |

Import them with **Skills → New → Import skills from project**, and re-run that after they change.

---

## Part 3: API reference

Observed on 2026.1023.0 in `local_trusted` mode, where browser requests to `http://localhost:3100` need no auth. `{cid}` is the company id, from `GET /api/companies?scope=accessible`.

### Agents

| Call | Notes |
|---|---|
| `GET /api/companies/{cid}/agents` | List agents |
| `GET /api/agents/{id}` | Full config: `adapterConfig`, `runtimeConfig` |
| `PATCH /api/agents/{id}` | Send `name`, `title`, and the **whole** `adapterConfig` / `runtimeConfig` objects (read, modify, send back) |
| `adapterConfig.model`, `.effort`, `.engine` | e.g. `claude-opus-5-5`, `high`, `cli` |
| `adapterConfig.env` | `{ "KEY": { "type": "plain", "value": "…" } }` |
| `adapterConfig.paperclipSkillSync.desiredSkills` | Array of skill **keys** (not ids) |
| `runtimeConfig.heartbeat.maxConcurrentRuns` | Set to `1` |

### Agent instructions

| Call | Notes |
|---|---|
| `GET /api/agents/{id}/instructions-bundle` | Lists files |
| `GET /api/agents/{id}/instructions-bundle/file?path=AGENTS.md` | Returns `content` and `revision.id` |
| `PUT /api/agents/{id}/instructions-bundle/file` | Body `{ path, content, baseRevisionId }`. `baseRevisionId` is the `revision.id` you just read. Old versions stay as revisions. |

### Skills

| Call | Notes |
|---|---|
| `GET /api/companies/{cid}/skills` | List, with `key` and `attachedAgentCount` |
| `POST /api/companies/{cid}/skills` | Create: `{ name, slug, description, markdown }` |
| `PATCH /api/companies/{cid}/skills/{id}/files?path=SKILL.md` | Edit: `{ path, content }` |
| `DELETE /api/companies/{cid}/skills/{id}` | Remove |
| `POST /api/companies/{cid}/skills/import` | `{ source }`, a GitHub URL |
| `POST /api/companies/{cid}/skills/scan-projects` | Imports `skills/*/SKILL.md` from project workspaces |

### Issues and runs

| Call | Notes |
|---|---|
| `GET /api/companies/{cid}/issues?limit=100&view=compact` | List |
| `POST /api/companies/{cid}/issues` | `{ title, description, projectId, assigneeAgentId, status, priority }` |
| `PATCH /api/issues/{id}` | e.g. `{ status: "done" }` |
| `GET` / `POST /api/issues/{id}/comments` | Post `{ body }` |
| `GET /api/companies/{cid}/heartbeat-runs?limit=20&summary=true` | Recent runs, with `status`, `error`, `issueId` |
| `GET /api/companies/{cid}/live-runs` | What's running now |
