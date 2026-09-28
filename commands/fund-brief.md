---
description: One-page client brief on one Tuatara-managed fund from cymetica.com, with as-of stamp, status, return since inception and the platform's disclosures
argument-hint: <fund symbol, e.g. MA-RIPPLE or AIB-ROBOTICS>
---

Write a one-page client brief on the fund `$ARGUMENTS` using the
`eventtrader-funds` tools. Follow the `fund-research` skill's rules for how
to present the numbers.

1. Call `get_fund_methodology` and keep its rules in view.
2. If no symbol was given, or `get_fund` returns `TOOL_NOT_FOUND`, call
   `list_funds` and ask which fund to brief, listing the symbols it returned.
   Don't guess a symbol.
3. Call `get_fund` with the symbol. If the fund has a history series, call
   `get_fund_history` with `points: 60` for the trend line. `FTA-STOCKS` and
   `EVENT-CARD` have none, so skip that call for them.

Write the brief in this order, in plain prose, under 400 words:

- **Heading**: fund name, symbol, and "as of <as_of timestamp>".
- **Status**: `live`, `paper` or `standing`, with what the status means.
- **Mandate**: theme, direction (long or short book) and mandate, in the
  fund's own terms. Give deployment as percentages only.
- **Performance**: return since inception (sign already corrected for short
  books) and, if you pulled history, one sentence on the trend from inception
  to the latest point. Call `index_price_usd` the index price.
- **What is withheld**: say which constituents are aliased or withheld by
  the platform. Don't guess names.
- **Disclosures**: quote the response's `page_note` verbatim, say that this
  is information, not a recommendation, and that suitability is the
  advisor's call.
- **Links**: the fund's `profile_url` and https://cymetica.com/fund-performance.
