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

If upgrading from version 0.2.x, remove its legacy Codex hooks once:

```bash
herdr plugin action invoke dev.herdr-codex-cost.remove-hooks
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

Restart Herdr or reload its configuration.

## Cost refresh

When Herdr reports a Codex pane as `working`, the plugin refreshes it every five
seconds. Polling stops as soon as the pane leaves `working`, and the status
change and polling worker each ensure a final refresh. During a quick session
restart, the last calculated cost remains visible until the new session has
usage to replace it.

Each pane caches its session-tree fingerprint from rollout paths, file sizes,
and modification times. During polling, an unchanged fingerprint reuses the
cached cost without parsing token events or updating sidebar metadata.

You can refresh the currently focused pane manually at any time:

```bash
herdr plugin action invoke dev.herdr-codex-cost.refresh
```

## Display values

- `~$1.23` means the available usage was calculated successfully.
- `!!$1.23` means some usage was excluded because of an unknown model, damaged log, or missing descendant session.
- `~$1.23(+$0.45)` separates the unofficial estimate for `codex-auto-review` from costs based on published prices.
- `session?` means Herdr could not associate the pane with a Codex thread ID.
  Per-pane diagnostics are written to
  `${TMPDIR:-/tmp}/herdr-codex-cost-$UID/error-*.log` and removed automatically
  after the association recovers. During a quick session restart, an existing
  calculated cost is retained instead of being replaced with `session?`.

## Testing

```bash
./test/codex-cost-test
./test/codex-hook-test
./test/poll-cost-test
./test/update-cost-test
```

## Publishing

Add the `herdr-plugin` topic to a public GitHub repository to make it discoverable in the Herdr marketplace. Users can then install it with `herdr plugin install owner/repo`.
