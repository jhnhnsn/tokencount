# tokencount

A Claude Code plugin that shows your Claude plan, session (5-hour) and weekly usage limits,
reset times, burn rate and token counts, and can stop Claude Code when usage crosses a threshold.

```
Session (5-hour window)
  ██████░░░░░░░░░░░░░░░░░░░░░░░░  20% used · 80% left
  Resets   14:50 PDT (in 2h 10m)
  Pace     on track for ~64% by reset at 2.8× current burn
  Tokens   4.0M fresh (in 1.6k · out 1.6M · cache-write 2.4M) + 70.6M cache-read · 763 requests
```

In Claude Code, just ask:

- "How much of my session is left?" / "When does my weekly limit reset?"
- "Stop when half my tokens remain" / "Don't go past 80%" / "Leave me 20% of the week"
- "Clear the usage guard"

## Install

For yourself (all projects):

```sh
claude plugin marketplace add jhnhnsn/tokencount
claude plugin install tokencount@tokencount
```

For everyone working in a project, commit this to the project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "tokencount": { "source": { "source": "github", "repo": "jhnhnsn/tokencount" } }
  },
  "enabledPlugins": { "tokencount@tokencount": true }
}
```

Requires Python 3 and a Claude Code login with a Pro or Max subscription. With an API key,
plan limits show as unavailable and only local token counts work.

## How it works

| Piece | Role |
|---|---|
| `tokencount` | Python script (stdlib only). Also usable directly from a shell. |
| `SKILL.md` | Teaches Claude to answer usage questions and map requests to `guard` commands. |
| `hooks/hooks.json` | `PreToolUse` hook running `tokencount guard check` before every tool call. |

Data comes from the OAuth token Claude Code stores in `~/.claude/.credentials.json`, the
undocumented `api.anthropic.com/api/oauth/usage` and `/profile` endpoints that `/usage` uses
(read-only `GET`s), and token counts in `~/.claude/projects/**/*.jsonl`. Responses are cached in
`~/.cache/tokencount/` so a transient API error falls back to data up to 30 minutes old.

Anthropic reports limits as percentages, not token counts. **Pace** projects the current window to
its reset, scaled by the **burn rate**: fresh tokens/min over the last 15 minutes compared with the
median of your active 15-minute blocks over the past week.

### The guard

```sh
tokencount guard set --remaining 50            # session window by default
tokencount guard set --used 80 --window week   # or a model limit, e.g. --window fable
tokencount guard status
tokencount guard clear
```

Once the window crosses the threshold, the hook blocks every tool call (except `tokencount`
itself) and tells Claude to stop and report. Limits are account-wide, so the guard applies to all
Claude Code sessions with the plugin. It expires when that window resets unless set with
`--persist`. Usage percentages move in coarse steps and are rechecked at most once a minute, so the
guard can trip slightly past the threshold. If usage can't be read, the hook lets calls through.

## Development

```sh
claude --plugin-dir .                       # try local changes in a session
claude plugin validate .
```

An installed plugin is a cached copy: bump `version` in `.claude-plugin/plugin.json`, then run
`claude plugin marketplace update tokencount && claude plugin update tokencount@tokencount`.
