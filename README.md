# Lians check

Every agent PR, checked against the spec before it touches hardware.

Lians reads the ticket a pull request closes (as it reads now, including the discussion) and posts one comment:
each requirement marked met, missed, partly done or can't tell, with what changed since the work started
listed first. When someone edits the ticket, every open PR that closes it is checked again.

It's built for robotics and embodied-AI teams whose PRs are written by Claude Code, Codex, Cursor, Devin or
Copilot's coding agent: ROS 2 nodes, controllers, perception, sim configs. Specs move with the hardware, agents
build the version they started from, and reviewers read the diff, not the ticket. The limits, stops and
interfaces at the bottom of a ticket are exactly what gets dropped. It works the same for any team that merges
agent-written PRs.

What it looks like on robot code: [lians.ai/robotics](https://www.lians.ai/robotics) · A real check:
[demo PR](https://github.com/ebeirne/lians-check-demo/pull/2)

## Install (2 minutes)

1. Add a repository secret `ANTHROPIC_API_KEY` (Settings → Secrets and variables → Actions). Lians runs in
   your own Actions under your key; a review usually costs $0.10–0.30.
2. Add `.github/workflows/lians-check.yml`:

```yaml
name: Lians check
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]
  issues:
    types: [edited]
permissions:
  contents: read
  pull-requests: write
  issues: write
concurrency:
  group: lians-check-${{ github.event.pull_request.number || github.event.issue.number }}
  cancel-in-progress: true
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: ebeirne/lians-check@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

3. Open a PR that says `Fixes #123`. The comment appears within a few minutes.

Optional inputs: `linear-api-key` (read-only, to check Linear tickets named in the PR title, branch or body),
`fail-on-missed: "true"` to fail the check when a requirement is missed.

## What it will and won't say

Lians only grades issues the PR says it closes. It treats the ticket author's and maintainers' latest comments as
the current scope, and it says "can't tell" rather than "missed" when the work could live outside the diff or
the PR says it was done elsewhere. It does not review code quality; it answers one question: does this PR do
what the ticket asks?

## Pricing

Free for open-source projects, for a 30-day evaluation, and for design partners. Commercial teams after that:
$300/month per organisation, or $99/month for life for the first 10 teams who start before Dec 31, 2026 (code
`FOUNDING99`). See [lians.ai](https://www.lians.ai).

Questions or noise reports: ethan.g.beirne@gmail.com
