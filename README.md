# herdr-codex-cost

English | [한국어](README.ko.md)

A Herdr plugin that calculates the estimated API-equivalent cost of the Codex session running in each pane, including its descendant agents, and displays it in the Agent sidebar.

The calculation is based on the model pricing and aggregation behavior from `tmux-statusline`'s `codex-session-usage`. The displayed amount is an estimate using public Standard API prices, not the actual amount charged for a ChatGPT or Codex subscription.

## Model pricing

Rates follow the [official Standard API price list](https://developers.openai.com/api/docs/pricing),
checked on 2026-10-20. Newly supported model rates are USD per million tokens:

| Model | Input | Cache read | Cache write (30m) | Output |
| --- | ---: | ---: | ---: | ---: |
| GPT-6.1 Sol | $2.00 | $0.10 | $2.50 | $10.00 |
| GPT-6 Sol | $2.00 | $0.20 | $2.50 | $10.00 |
| GPT-6 Luna | $0.10 | $0.01 | $0.125 | $0.50 |
| GPT-5.6 Cyber | $12.50 | $1.25 | $15.625 | $75.00 |
| GPT-5.5 Cyber | $12.50 | $1.25 | Ordinary input rate | $75.00 |

GPT-6 Astra and existing GPT-5.6 models remain supported. Daybreak blue/red
aliases use GPT-5.6 Sol/Cyber rates respectively. For GPT-6.1 Sol, GPT-6, GPT-5.6 Sol/Terra/Luna,
GPT-5.5 and GPT-5.4 base/Pro models, requests above 272,000 input tokens use
2x input/cache rates and 1.5x output rates.
[Cache writes](https://developers.openai.com/api/docs/guides/prompt-caching)
on supported models use the 30-minute rate for legacy `cache_write_input_tokens` counters.
When rollout usage preserves the official Responses API
`input_tokens_details.cache_write_tokens_details` breakdown, `ttl_30m_tokens`
and `ttl_12h_tokens` are charged at 1.25x and 2.5x input respectively, including
long-context multipliers. The components replace ordinary input and are not
counted again as total writes. Logs without this breakdown retain the 30-minute
estimate; 12-hour or mixed-lifetime costs cannot be recovered from a total alone.
Malformed or conflicting breakdowns are excluded and flagged with `!!`.
Historical logs are also valued at current rates; Fast/Ultrafast/Batch/Flex rates and subscription credit multipliers do not apply.

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

- `$1.23` means the available usage was calculated successfully.
- `!!$1.23` means some usage was excluded because of an unknown model, damaged log, or missing descendant session.
- `$1.23(+$0.45)` separates the unofficial estimate for `codex-auto-review` from costs based on published prices.
- `session?` means Herdr could not associate the pane with a Codex thread ID.
  Per-pane diagnostics are written to
  `${TMPDIR:-/tmp}/herdr-codex-cost-$UID/error-*.log` and removed automatically
  after the association recovers. During a quick session restart, an existing
  calculated cost is retained instead of being replaced with `session?`.
  When Herdr's association is unavailable, the plugin reads the thread ID
  prefix from the Codex pane status line and checks it against rollout IDs and
  working directories. The fallback is accepted only when exactly one rollout
  matches; otherwise `session?` is retained.
  If no ID is visible, it matches the terminal title against session names and
  working directories in Codex's state database, then verifies the unique match
  against rollout metadata. This also supports shared app-server sessions whose
  hooks do not inherit the pane environment.

## Testing

```bash
./test/codex-cost-test
./test/codex-hook-test
./test/poll-cost-test
./test/update-cost-test
./test/resolve-pane-session-test
```

## Publishing

Add the `herdr-plugin` topic to a public GitHub repository to make it discoverable in the Herdr marketplace. Users can then install it with `herdr plugin install owner/repo`.
