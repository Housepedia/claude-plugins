# Changelog

> What changes. Within v5, fields and endpoints are only added.

Source: https://huispedia.dev/en/docs/changelog/

Within v5, fields and endpoints are only added; what you use now keeps working. See [Versioning](https://huispedia.dev/en/docs/versioning/).

## 2026-11

- **New, with a contract:** three sustainability endpoints (contact sales@huispedia.nl). `GET /properties/{locator}/energy-profile` gives the energy label (registered or estimated), the solar potential of the roof, the energy use of similar homes in the neighbourhood and the current tariffs in one call; `GET /properties/{locator}/solar-potential` gives panels, capacity and yearly yield per home, per building and, with `include=planes`, per roof plane; `GET /areas/{area}/energy` gives energy use, solar panels and heating per neighbourhood, district and municipality.
- **New, with the same contract:** `GET /properties/{locator}/energy-label` adds the energy figures of the registered label (`registered`) and, for a home without a registered label, an estimated label under `estimate`. `label` never holds an estimate. With the contract, a home without a registered label answers 200 with the estimate where it used to answer 422. `GET /buildings/{bag_pand_id}` adds the building's 3D shape (`shape`). Without the contract both endpoints answer as before.

## 2026-10

- **New:** the v5 API with 21 endpoints: addresses, property characteristics, energy label, WOZ, building, condition, transactions, valuation, valuation plus, reference homes, valuation history, scenario valuations, areas, statistics, price index, demographics, surroundings and environment.
- **New:** the dashboard: sign in with your huispedia.nl account, keys, balance, usage and request log.
