# Authentication

> How to send your API key.

Source: https://huispedia.dev/en/docs/authentication/

Send your key in the header `X-API-Key: hp_…` or `Authorization: Bearer hp_…`, never in the URL.

- **You see a key once.** Lost it? Create a new one and revoke the old one.
- **Per key** you can set an expiry date and an IP allowlist.
- **A key belongs to one account**: your own, or a Pro organisation you manage. Switch in the dashboard.
- **`console`** in your request log is the temporary key of the endpoints page and the quickstart.

When your key does not work you get a `401` or `403`; the [error codes](https://huispedia.dev/en/docs/errors/) say why.
