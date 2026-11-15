---
name: paperclip-team-reporting
description: >
  How every agent in a Paperclip coding team writes comments and handovers: the three-line
  update on a ticket's parent issue that the owner reads, the short handover on Build, Test
  and Review issues, when to post at all, and why to write nothing between tool calls. Use
  before posting any comment.
---

# Reporting

The owner reads the parent issue of each ticket and nothing else. Write for someone who has ten seconds.

## Everything you write can become a comment

Paperclip can post any text you write during a run as a comment, not only your last message. So write no text between tool calls: no "I'll look up…", no "The run is in progress", no progress notes. Write once, at the end, in one of the shapes below. Never a summary of your run.

## On the parent issue (Lead only)

Post only when one of these is true:

- The owner needs to decide something. Use a question card, not a paragraph.
- A pull request is open and ready to merge.
- Work is stuck on something only the owner can fix.
- A follow-up was filed: one line, `Follow-up: ABC-123 · <title>`.
- The ticket is finished.

Use exactly this shape:

```
Status: ABC-12 · PR open → #42
Needs you: Merge #42
Next: Nothing until you merge
```

- `Needs you:` says `Nothing` when the owner has nothing to do.
- If an agent or Paperclip itself is failing, name it and what fixes it on the `Needs you:` line.
- When the owner asks what's happening, answer with this block and nothing else.
- When you only routed work, your final message is one line: `Routed: Build → Dev`.
- No history, no apology, no account of what each agent did. Link the child issue instead.

Never post "starting", "still working", "waiting on X" or "nothing changed". Silence means work is moving.

## On Build, Test and Review issues (all agents)

A handover is at most 15 lines, bullets only:

```
Commit: abc1234 on task/ABC-12
Result: type-check ✓ · unit 31/31 · e2e 42/42
Changed: <files or areas, one line each>
Not checked: <one line each, or none>
Next: <who acts, and what>
```

Add one extra bullet for detail that matters later, such as a root cause or a decision a reviewer might question. Logs, command output and file-by-file narration stay in the run log.

## Handing an issue to someone else

Paperclip ends your run the moment an issue you are running on changes assignee. Anything after that is lost, including your handover. So, in this order:

1. Post your handover as a comment.
2. Finish everything else.
3. Reassign the issue as your very last action, with nothing after it.

## Never

- Repeat what another agent already posted
- Restate the task description or the repo's rules
- Post the same update on more than one issue
- Use headings or tables in a comment
