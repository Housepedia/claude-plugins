# Credits and pricing

> What a call costs and which credits go first.

Source: https://huispedia.dev/en/docs/credits/

Every endpoint costs a fixed number of credits. A credit is €0.05 excluding VAT; on the Free plan every user account gets 200 free credits a month.

Every endpoint with its credits, fields and sources is in the catalogue: [`GET https://huispedia.dev/v5/catalog`](https://huispedia.dev/v5/catalog) (JSON, no key needed) and the [OpenAPI document](https://huispedia.dev/openapi.json).

**What costs and what does not.** Only a successful answer (2xx) costs credits. A 404, a validation error or an outage costs nothing, and so does `HEAD`. In a scenario valuation you pay per successful row and per scenario.

**Which credits first.** Your free credits first, then your plan's, then your bought credits, oldest first. Bought credits are valid for 12 months; the dashboard shows each bundle's end date.

**Spend cap.** 2,000 bought credits (€100 excl. VAT) a month by default, on top of your free credits and your plan. Raise it under *Settings*; at the cap the API answers `402 spend_cap_reached`.

**Transactions.** Transaction prices come with a contract; contact [sales@huispedia.nl](mailto:sales@huispedia.nl). Kadaster transaction records in an answer count in `X-Kadaster-Records` and appear as their own line on your monthly statement.

## Plans

A paid plan gives you credits every month for a fixed price, cheaper than one by one. Plan credits you do not use expire at the end of the month. Credits you buy on top always cost €0.05 excluding VAT and are valid for 12 months, plan or not. A bigger discount comes with a bigger plan.

What an endpoint costs in credits is in the catalogue. If that changes, you hear it by email at least 30 days ahead.

The plans with their prices and credits are under `packs` in [`GET https://huispedia.dev/v5/catalog`](https://huispedia.dev/v5/catalog).
