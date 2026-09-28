# EventTrader Funds — connector documentation

Read-only research on every Tuatara-managed fund at cymetica.com, for
financial-advisor assistants (Claude for Financial Advisors, Claude Cowork,
claude.ai custom connectors, Claude Code). The connector returns exactly what
the public page https://cymetica.com/fund-performance shows, and nothing the
page withholds.

- Remote MCP server: `https://cymetica.com/mcp/v1/funds` (Streamable HTTP, JSON-RPC)
- Authentication: **none** for the fund tools
- Support: contact@cymetica.com
- Privacy policy: https://cymetica.com/privacy · Terms: https://cymetica.com/terms

## Setup

**Claude for Financial Advisors / Cowork / claude.ai** — add a custom
connector with the server URL above. No login, no key.

**Claude Code**

```
/plugin marketplace add BioVector/eventtrader-funds-plugin
/plugin install eventtrader-funds@cymetica
```

or point Claude Code at the server directly:

```
claude mcp add --transport http eventtrader-funds https://cymetica.com/mcp/v1/funds
```

## Tools

All four are read-only (`readOnlyHint: true`). They take no credentials and
touch no user data.

| Tool | What it returns |
|---|---|
| `get_fund_methodology` | The measurement rules: as-of, index construction, return convention, status meanings, what is withheld, execution venue, links. Call once per session. |
| `list_funds` | The ranked roster (best return since inception first). Filter by `status`: `live`, `paper`, `standing`, `all`. `limit` 1–100. |
| `get_fund` | One fund by `symbol` (an AIB symbol such as `AIB-BIOSCIENCES`, or a sleeve: `MA-RIPPLE`, `FTA-CRYPTOS`, `FTA-STOCKS`, `EVENT-CARD`). |
| `get_fund_history` | Index history for one fund as `[{t, v}]` points, `points` 2–400, always starting at inception and ending at the latest point. AIB symbols, `MA-RIPPLE` and `FTA-CRYPTOS` only — `FTA-STOCKS` and `EVENT-CARD` have no history series; use `get_fund` for their current numbers. |

## Example prompts

- "Which Cymetica funds are live, and how has each done since inception?"
- "Pull the M&A Ripple fund and summarise it for a client note, as of today."
- "Chart the last 60 points of AIB-BIOSCIENCES and describe the trend."
- "What does Cymetica withhold from these numbers, and why?"

## What the numbers mean

- `return_since_inception_pct` is the **fund** return. For a short book a
  falling index is a positive return; the sign is already corrected.
- `index_price_usd` is a price-weighted **index price**, not a share price
  and not a capital figure. Capital, AUM and allocations are never published;
  the mandate and deployment are given as percentages.
- `status`: `live` takes real USDC through https://cymetica.com/fund/apply;
  `paper` trades simulated and takes no investment; `standing` is live but
  not yet allocated.
- Short-book constituents are aliased and open M&A ripple baskets are
  withheld while positions are open. The execution venue is referred to as
  **Brokerage100**.
- Every response carries an `as_of` stamp. Quote figures as of that time.
- Nothing here is investment advice. The Tuatara Hedge Fund is described by
  the platform as highly experimental; suitability is the advisor's call.
  Every response carries `page_note`, the note on
  https://cymetica.com/fund-performance, verbatim: "This fund is highly
  experimental. Every figure on this page is a live measurement, not a track
  record."

## Limits

Each client address gets 120 requests per minute, 3,000 per hour and 30,000
per day. Fund data is served from a short-lived cache, so a burst of advisor
queries costs one build. Over the limit the server answers with a JSON-RPC
error (`RATE_LIMITED`) that says how many seconds to wait before retrying.

## Errors

A failed call comes back as an MCP tool result with `isError: true` and a
JSON body naming a code and a readable message:

- `MISSING_PARAMETER` / `INVALID_PARAMETER_TYPE`: an argument is missing or
  the wrong type, e.g. "Missing required parameter: symbol".
- `TOOL_NOT_FOUND`: an unknown fund symbol ("call list_funds"), or a history
  request for `FTA-STOCKS` / `EVENT-CARD`, which have no series.
- `INVALID_PARAMETER_TYPE` from `list_funds`: `status` is not one of
  `live`, `paper`, `standing`, `all`.
- `TOOL_ERROR`: fund data is temporarily unavailable; retry shortly.

A history request can also succeed with no points; the response then carries
a `note` saying so. No internal detail is ever returned.

## What this plugin runs, sends and fetches

- It runs nothing on your machine: no hooks, no scripts, no local server, no
  package installs.
- It connects to exactly one server, `https://cymetica.com/mcp/v1/funds`,
  and nothing else. That endpoint serves only the four tools above; every
  other tool name is refused, and it exposes no resources.
- Each call sends the tool name, its arguments (a fund `symbol`, a `status`
  filter, a `limit` or a `points` count) and the header
  `User-Agent: eventtrader-funds-plugin/1.1`. It sends no credentials,
  conversation text, files or account data.
- It cannot place orders, move money, open accounts or change anything on
  cymetica.com. Links it returns (for example the fund application page)
  are for a person to open.

## Privacy

The fund tools read public fund data only. They do not receive, store or
log any user data, conversation content, or account information. Server
access logs keep the requesting IP and user agent for security, under the
policy at https://cymetica.com/privacy.

## Data ownership

The API and the data are Cymetica's own. Underlying market prices come from
the execution venues the platform trades on; the index arithmetic is the
platform's.

## Bundle contents

- `.claude-plugin/plugin.json`: the manifest
- `.claude-plugin/marketplace.json`: lets Claude Code install the plugin
  straight from this repository
- `.mcp.json`: the remote MCP server
- `skills/fund-research/SKILL.md`: how to research a fund and present the
  numbers to a client
- `commands/fund-brief.md`: `/eventtrader-funds:fund-brief <symbol>`, a
  one-page client brief on one fund
- `evals/`: `claude plugin eval` cases for the plugin
- `LICENSE`: MIT
