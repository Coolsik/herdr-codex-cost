# herdr-codex-cost

English | [한국어](README.ko.md)

A Herdr plugin that calculates the estimated API-equivalent cost of the Codex session running in each pane, including its descendant agents, and displays it in the Agent sidebar.

The calculation is based on the model pricing and aggregation behavior from `tmux-statusline`'s `codex-session-usage`. The displayed amount is an estimate using public Standard API prices, not the actual amount charged for a ChatGPT or Codex subscription.

## Requirements

- Herdr 0.7.0 or later
- `python3` and `jq`
- The Codex integration (`herdr integration install codex`)

## Installation

Install directly from GitHub:

```bash
herdr plugin install Coolsik/herdr-codex-cost
herdr integration install codex
```

For local development, link the checkout instead:

```bash
herdr plugin link "$PWD"
herdr integration install codex
```

## Sidebar configuration

Add `$cost` to the Codex Agent rows in `~/.config/herdr/config.toml`:

```toml
[ui.sidebar.agents.rows_by_agent]
codex = [
  ["state_icon", "agent", "$cost"],
  ["workspace", "tab"],
]
```

Restart Herdr or reload its configuration. The cost is refreshed whenever the Codex agent status changes. You can also refresh the currently focused pane manually:

```bash
herdr plugin action invoke dev.herdr-codex-cost.refresh
```

## Display values

- `~$1.23` means the available usage was calculated successfully.
- `!!$1.23` means some usage was excluded because of an unknown model, damaged log, or missing descendant session.
- `~$1.23(+$0.45)` separates the unofficial estimate for `codex-auto-review` from costs based on published prices.

## Testing

```bash
./test/codex-cost-test
```

## Publishing

Add the `herdr-plugin` topic to a public GitHub repository to make it discoverable in the Herdr marketplace. Users can then install it with `herdr plugin install owner/repo`.
