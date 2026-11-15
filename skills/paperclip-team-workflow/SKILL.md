---
name: paperclip-team-workflow
description: >
  How a Paperclip lead agent turns a ticket into issues and moves it through build, test,
  review and pull request without stalling: the fixed issue shape and titles, handing work
  over by assignment instead of blockers, fix rounds, follow-ups and closing. Use when
  planning a ticket or routing work between Dev, QA and Review.
---

# From ticket to pull request

## The shape, always

```
ABC-12 · Short ticket title       Lead     ← parent, the only issue the owner watches
 ├ ABC-12 · Build                 Dev
 ├ ABC-12 · Test                  QA
 └ ABC-12 · Review                Review
```

- Every title starts with the ticket key and ` · `. Children are exactly `Build`, `Test` and `Review`.
- Never more than two levels. Never a child under Build, Test or Review.
- With no ticket key, use a short topic in its place: `Docs cleanup · Build`.

## Planning

1. Read the ticket, the repo's agent guidance (`AGENTS.md` or similar) and only the docs for the areas the ticket touches. Do this once, when planning, not on every wake.
2. Questions for the owner go in **one question card** on the parent, all at once. A question in a plain comment gives you nothing to wait on.
3. Put the whole brief in the **Build** issue: goal, acceptance criteria, likely files, rules at risk, and the branch (`task/ABC-12` off the base branch). Test and Review point to it rather than repeating it.
4. Pin Test and Review to Build's execution workspace, so all three share one worktree and branch. A review in a fresh workspace reviews an empty branch.
5. One ticket is one pull request into the base branch. Never stack branches.

## Moving work

Hand work over by **assignment**, never by blockers.

- Never set a blocker on the parent, and never between Build, Test and Review. Paperclip won't wake the assignee of a blocked issue, so a handover onto a blocked issue stalls silently.
- Each agent finishes by commenting on its own issue, then assigning it back to Lead as its very last action. Paperclip ends the run on reassignment, so anything after it is lost.
- Lead reads the handover and assigns the next issue: Build → Test → Review.
- Only one agent works on the branch at a time.

## CI runs the full suite, not the agents

No agent runs the full end-to-end suite locally. It is slow, uses a large share of the model usage limit, and a shared test database makes it fragile. CI runs it on every push to the pull request.

1. **Build:** Dev builds, runs type-check and unit tests, pushes and opens a **draft** pull request into the base branch, so CI runs.
2. **Test:** QA writes specs for the acceptance criteria, runs **only those** locally, and pushes them. For UI work QA also does a visual pass, which CI can't.
3. **Review:** Review checks the branch's head commit and the CI result for that exact commit. It approves only when CI is green on it.
4. **Ready:** Lead marks the draft ready for review and posts the three-line update. Only now does it reach the owner.

## Results belong to a commit

- Every handover names the commit it was produced on.
- Review approves only a head commit CI passed. If CI is still running, it waits.
- If the base branch moved, Dev merges it in (never rebases) and pushes. CI re-tests the merged result, so nobody re-runs anything by hand.

## Fix rounds

When Review or CI finds a problem:

1. Copy the finding word for word into a comment on **Build** and reassign Build to Dev. Add your decision on each optional item in one line.
2. Dev fixes and pushes. CI re-runs on its own.
3. Reassign **Review** for the new head commit. Reassign **Test** only if the specs need to change.

Reopen the same three issues every round. Never create new Build, Test or Review issues.

## The pull request

The draft opened in Build is the ticket's pull request. Never open a second one. Once Review approves the head commit and CI is green on it, Lead marks it ready for review, posts the link on the parent with the three-line update from `paperclip-team-reporting`, then marks Build, Test and Review done.

## Follow-ups

Follow-ups go in the issue tracker the owner plans from (Jira, Linear, GitHub Issues), not in Paperclip. Anything out of scope that an agent finds, such as a bug, a gap or a stale doc, becomes a ticket there. Lead creates it.

- **Title:** what is wrong, in plain words.
- **Description:** where it is (`file:line`), how it was found, and what done looks like. Link the ticket it came from.
- Then post one line on the parent: `Follow-up: ABC-123 · <title>`.
- Never start work on it. The owner decides when it runs, and it comes back as a new ticket.

Paperclip holds only work in progress. Nothing waits in its backlog. If the owner doesn't use a separate tracker, file the follow-up as a top-level Paperclip issue and list it on the parent, so it isn't buried under a finished ticket.

## Closing

When the owner merges, clean up before anything else:

1. Archive the ticket's execution workspace in Paperclip (status `archived`).
2. `git worktree remove --force <path>` on its folder, then `git worktree prune`.
3. `git branch -d <branch>`. Use `-d`, never `-D`: git refuses an unmerged branch, and that refusal is the safety check.
4. Mark Build, Test, Review and the parent done. Don't post on the parent again.

Clean up only after a merge, never before. A weekly routine removes anything this step missed.

Agents work only in the workspace Paperclip gives an issue. Nobody runs `git worktree add` by hand, because Paperclip can't clean up a worktree it didn't create.
