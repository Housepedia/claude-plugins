# Quickstart

> From signing in to your first home value.

Source: https://huispedia.dev/en/docs/quickstart/

1. Sign in at [https://huispedia.dev/en/dashboard/signin/](https://huispedia.dev/en/dashboard/signin/) with your huispedia.nl or Pro account.
2. Choose your plan; Free is fine.
3. Create an API key (`hp_…`) under *API keys*. You see it once.
4. Value a home:

```bash
curl https://huispedia.dev/v5/properties/1076LJ-12-2/valuation \
  -H "X-API-Key: $HUISPEDIA_KEY"
```

<details class="hp-sample">
<summary>What the answer looks like</summary>

```json
{
  "address": {
    "bag_id": "0363200000028422", "postcode": "1076LJ", "house_number": 12, "house_number_ext": "2",
    "street": "Amazonenstraat", "city": "Amsterdam", "municipality": "Amsterdam", "province": "Noord-Holland",
    "location": { "lat": 52.34536032, "lon": 4.86274012 },
    "huispedia_url": "https://huispedia.nl/amsterdam/1076LJ/amazonenstraat/12-2"
  },
  "valuation": { "value": 888264, "lower": 838099, "upper": 941431, "interval": 0.8, "price_per_m2": 9064, "as_of_date": "2026-10-26", "model": "avm4", "model_version": "2026.10" },
  "confidence": { "score_pct": 90.3, "class": "high", "within_5_pct": 45.2, "within_10_pct": 90.3, "within_20_pct": 100.0 },
  "meta": { "request_id": "req_01M5W27EFJTGG4DQDJ6DVK3NVY", "as_of": "2026-10-26", "credits": { "charged": 8, "balance": 192 } }
}
```

`value` is the Huispedia home value; the actual value lies between `lower` and `upper` with 80% certainty.

</details>

## Next

- [Data catalogue](https://huispedia.dev/en/docs/catalogue/): what else you can ask for.
- [Endpoints](https://huispedia.dev/en/docs/endpoints/): every endpoint, to try with your own credits once signed in.
- [Addresses and areas](https://huispedia.dev/en/docs/locators/): point at a home by BAG id or at an area by CBS code.
- [Migrating from Altum](https://huispedia.dev/en/docs/migrate/altum/)
- [Migrating from Matrixian](https://huispedia.dev/en/docs/migrate/matrixian/)
