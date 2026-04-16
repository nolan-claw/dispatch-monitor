# Dispatch Monitor

Monitor active dispatch tasks, check CI status, and track reconciliation progress.

## Checking Active Tasks

```bash
# Status board
ssh dev@178.156.171.0 dotfiles-dispatch-status

# Specific task log
ssh matchpoint@178.156.171.0 "tail -40 ~/ralph-state/TASK-ID.log"

# Task status JSON
ssh matchpoint@178.156.171.0 "cat ~/ralph-state/TASK-ID.status"

# Live tail
ssh matchpoint@178.156.171.0 "tail -f ~/ralph-state/TASK-ID.log"
```

## Status JSON Fields

```json
{
  "status": "running|done|failed|waiting",
  "iter": 3,
  "max": 30,
  "started": "2026-04-16T04:04:10Z",
  "finished": "2026-04-16T04:23:03Z",
  "marker_found": true,
  "stories_passed": 4,
  "stories_total": 4,
  "repo": "matchpoint"
}
```

## Monitoring Patterns

### Polling a Running Task
```bash
# Wait N seconds, then check
sleep 120 && ssh matchpoint@178.156.171.0 "tail -40 ~/ralph-state/TASK-ID.log && cat ~/ralph-state/TASK-ID.status"
```

### Check All Recent Tasks
```bash
ssh matchpoint@178.156.171.0 "ls -lt ~/ralph-state/*.status | head -5 | while read line; do file=\$(echo \$line | awk '{print \$NF}'); echo \"=== \$file ===\"; cat \$file; done"
```

### Check Open PRs
```bash
ssh matchpoint@178.156.171.0 "cd /home/matchpoint/repositories/matchpoint && git fetch origin && git log origin/main --oneline -10"
```

## Health Indicators

| Signal | Meaning | Action |
|--------|---------|--------|
| `marker_found: true` | Task completed | Check verification result |
| `marker_found: false, iter > 10` | Ralph is struggling | Check log for loops |
| `Verification FAILED` | Build fails on main checkout | Usually missing node_modules |
| `Verification PASSED` | Ready to merge | Check PR mergeability |
| `stories_passed == stories_total` | All PRD stories done | Verify acceptance criteria manually |
| Task stuck at `waiting` | Dependency not met | Check depends-on task |

## Stopping a Stuck Task

```bash
ssh matchpoint@178.156.171.0 "touch ~/.ralph-stop"
```

## Common Issues

- **Build fails on verification but passes in worktree**: Main checkout has stale node_modules. Ralph usually fixes by running `pnpm install`.
- **PR targets wrong branch**: Ralph defaults to `wave0-foundation`. Fix with `gh pr edit NUMBER --base main`.
- **Merge conflicts after rebase**: Ralph handles rebases automatically in iteration 2+.
- **Stale GitHub mergeability**: Force-push or empty commit to refresh.
