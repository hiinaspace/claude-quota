# claude-quota

Headless JSON access to Claude Code's subscription quota (the `/usage`
panel), including the per-model weekly bar (e.g. Fable) that isn't available
anywhere else programmatically.

## Why

There's no official programmatic access to this data today:

- [anthropics/claude-code#21943](https://github.com/anthropics/claude-code/issues/21943)
  — open feature request for `/usage --json` or a usage cache file.
- [anthropics/claude-code#45392](https://github.com/anthropics/claude-code/issues/45392)
  — request for API access to usage/quota, closed as not-planned.

The statusline input JSON *does* officially carry
`rate_limits.five_hour.used_percentage` and `rate_limits.seven_day.used_percentage`,
which is a complementary, fully-official partial alternative — but it's only
the overall pool, not the per-model weekly bars (like the Fable-specific one)
that `/usage` shows in the TUI.

## Example output

```json
{
  "session":    {"pct": 8,  "resets": "11pm (America/Denver)"},
  "week_all":   {"pct": 24, "resets": "Jul 8, 3am (America/Denver)"},
  "week_fable": {"pct": 41, "resets": "Jul 8, 3am (America/Denver)"},
  "captured_at": "2026-07-02T12:00:00-0600"
}
```

## Requirements

- Claude Code CLI, logged in to a subscription account
- GNU screen
- python3
- Linux/macOS

## Install

Copy `claude-quota` to `~/.local/bin` (or anywhere on `PATH`) and make sure
it's executable.

## How it works

It launches `claude` in a detached GNU `screen` session, sends `/usage`, and
scrolls to bring the per-model weekly bars into view. It then captures the
pane with `screen hardcopy` and parses the bars out of the text. A run takes
about 7 seconds and costs zero tokens/quota, since the `/usage` panel
populates without sending any API message. On first run it automatically
trusts its working directory (`~/.cache/claude-quota`) to get past Claude
Code's folder-trust prompt.

## Motivating use case

Lets an agent or script check remaining top-tier-model quota before deciding
which models to route subagent work to.

## Caveats

- It scrapes a TUI, so any Claude Code UI change can break it. It's
  deliberately loud on parse failure (dumps the raw capture to stderr, exits
  1) rather than silently returning stale or wrong numbers.
- Model-specific keys are derived generically from whatever labels the panel
  shows (`week_fable` today), so it should adapt automatically to future
  model names.
- Each run leaves a small empty session file under `~/.claude/projects/`.
- Percentages reflect the same approximations `/usage` itself shows in the
  TUI — this is a scrape, not a more-precise measurement.
