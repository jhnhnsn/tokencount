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
- "When does my weekly limit reset?"
- "Stop when half my tokens remain" / "Don't go past 80% of the week"
- "Clear the usage guard"

Or run the script directly: `tokencount`, `tokencount --json`, `tokencount guard --help`.

## Notes

- Limits come from the undocumented endpoint behind Claude Code's `/usage`, using your
  Claude Code login. Token counts come from your local Claude Code logs.
- A guard blocks tool calls in every Claude Code session on your account until you clear
  it or the usage window resets.

## License

[MIT](LICENSE)
