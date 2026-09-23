---
name: tokencount
description: Check the user's Claude plan, session (5-hour) and weekly usage limits, reset times, burn rate and token counts, and set a guard that stops work when usage crosses a threshold. Use when the user asks how much usage or how many tokens they have left, when limits reset, what plan they are on, or says things like "stop when half my tokens remain", "don't go past 80% of my session", "leave me 20% of the week", or "clear the usage guard".
allowed-tools: Bash(python3 "${CLAUDE_PLUGIN_ROOT}/tokencount" *)
---

# tokencount

`python3 "${CLAUDE_PLUGIN_ROOT}/tokencount"` reads the user's Claude subscription usage (the same
data as `/usage`) plus token counts from local Claude Code logs.

Limits are reported by Anthropic as **percentages** of each window, not token counts. When the
user says "tokens" they almost always mean these percentages; treat "half my tokens" as 50%.

## Checking usage

```sh
python3 "${CLAUDE_PLUGIN_ROOT}/tokencount" --json # parse this to answer questions
python3 "${CLAUDE_PLUGIN_ROOT}/tokencount"         # human-readable; show it when the user wants the overview
```

Key JSON fields: `account.plan`; `session` / `week` → `percent_used`, `percent_left`, `resets_at`
(UTC ISO, convert to local time), `pace.projected_percent` (projected use at reset),
`pace.hits_100_at`; `burn_rate.ratio` (current vs typical pace); `weekly_model_limits`;
`usage_source` (`live`, `cached` or `unavailable`, say so if not live).

Answer the question asked in a sentence or two. Don't paste the whole report unless asked.

## Stopping at a threshold (guard)

A PreToolUse hook enforces the guard. Once the window crosses the threshold, every tool call is
blocked for all Claude Code sessions on the account, since limits are account-wide.

```sh
tokencount guard set --remaining 50              # "stop when half my tokens remain"
tokencount guard set --used 80                   # "stop at 80%" / "don't go past 80%"
tokencount guard set --remaining 20 --window week   # "leave me 20% of the week"
tokencount guard set --used 90 --window fable    # a per-model weekly limit
tokencount guard status
tokencount guard clear
```
(`tokencount` above is shorthand: always run it as `python3 "${CLAUDE_PLUGIN_ROOT}/tokencount"`.)

Mapping requests:
- Default window is `session` (the 5-hour window). Use `week` when the user mentions the week or
  weekly limit. If it's truly ambiguous, pick session and say so in your reply.
- "X% remain / left / leave me X%" → `--remaining X`. "at X% / X% used / past X%" → `--used X`.
- The guard expires on its own when that window resets. Add `--persist` only if the user wants it
  to outlive the reset.
- After setting it, confirm in one line: window, threshold, current usage, expiry.

Usage percentages from the API move in coarse steps and are cached for up to a minute, so the
guard can trip slightly past the threshold. Mention this if the user needs a hard margin.

## When the guard trips

A blocked tool call returns a message starting `tokencount guard tripped`. Then:
1. Stop. Make no further tool calls, and don't retry or work around the block.
2. Tell the user what you finished, what remains, and the current usage.
3. Run `tokencount guard clear` only if the user explicitly asks to continue.
