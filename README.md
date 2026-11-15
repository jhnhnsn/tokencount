# tokencount

A Claude Code plugin that shows your Claude plan, session and weekly usage limits, reset
times and burn rate, and can stop Claude when usage crosses a threshold you set.

## Install

```sh
claude plugin marketplace add jhnhnsn/tokencount
claude plugin install tokencount@tokencount
```

Needs Python 3 and Claude Code signed in with a Pro or Max plan.

## Use

Ask Claude:

- "How much of my session is left?"
- "How much of the week have I used today?"
- "When does my weekly limit reset?"
- "Stop when half my tokens remain" / "Don't go past 80% of the week"
- "Hold me to a rolling daily cap of 14.3%"
- "Clear the usage guard"

Or run the script from a clone of this repo: `./tokencount`, `./tokencount --json`,
`./tokencount guard --help`.

## Notes

- Limits come from the undocumented endpoint behind Claude Code's `/usage`, using your
  Claude Code login (the credentials file, or the macOS Keychain). Token counts come from your
  local Claude Code logs.
- Today's share of the week is an estimate: the week's percentage scaled by today's share of
  the week's local tokens. Models weigh differently against the limit, and claude.ai and other
  devices aren't in the local logs.
- A guard blocks tool calls in every Claude Code session on your account until you clear
  it or the usage window resets.
- A rolling daily cap (`guard set --rolling 14.3 --window week`) lets day N of the usage week
  use N × 14.3% of it, so a light day leaves more for the next. Days are 24-hour periods from
  the week's reset time, not calendar dates. It stays until you clear it; `guard pause` lets
  work go on until the next day's limit starts.

## License

[MIT](LICENSE)
