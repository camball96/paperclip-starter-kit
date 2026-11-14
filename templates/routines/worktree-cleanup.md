# Routine: weekly worktree cleanup

A safety net for worktrees that the closing step of `paperclip-team-workflow` missed. Create it as a Paperclip routine assigned to Lead, with a weekly schedule trigger at a quiet hour, for example `41 7 * * 1` (Mondays 7:41am) in the owner's timezone.

**Title:** `Weekly · Clean up merged worktrees`

**Description** (this is the instruction Lead receives; fill in the placeholders):

```
Weekly cleanup of leftover worktrees and workspaces. Nothing to build. Read only this description.

1. In {{MAIN_CHECKOUT_PATH}}, run `git fetch origin`, then `git worktree list`.
2. For every worktree other than the main checkout whose branch is merged into
   `origin/{{BASE_BRANCH}}` (`git merge-base --is-ancestor <branch> origin/{{BASE_BRANCH}}`),
   or whose folder no longer exists:
   - archive its Paperclip execution workspace (status `archived`)
   - run `git worktree remove --force <path>`
   - delete the local branch with `git branch -d <branch>`. Use `-d`, never `-D`.
3. Run `git worktree prune`.
4. Never touch a worktree whose branch is unmerged, or one an active issue is using.
   List those instead.
5. Close this issue with one line:
   `Removed: <n> worktrees (<branches>) · Kept: <branches, or none>`. Post nothing else.
```

The API calls to create it are in [`SETUP.md`](../../SETUP.md#workspaces-and-routines).
