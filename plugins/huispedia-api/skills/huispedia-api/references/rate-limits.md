# Rate limits

> Calls per second per plan, and how long you may keep an answer.

Source: https://huispedia.dev/en/docs/rate-limits/

| Plan | Per second | At a time | Daily signal value |
|---|---|---|---|
| Free | 1 call, burst 5 | 2 | 500 calls |
| Starter, Growth, Scale, Pro | 6 calls, burst 20 | 6 | 20,000 calls |
| Enterprise | Per agreement | Per agreement | Per agreement |

The per-second limit is for your whole account, all your keys together. With a running plan or a purchase in the last 12 months you have the paid plans' limits. Free is for building and testing your integration.

The signal value is not a hard limit. If your calls stay above it for a while, we get in touch and look for a plan that fits together ([fair use](https://huispedia.dev/en/voorwaarden/#annex-b-fair-use-policy), Annex B).

Every answer carries `X-RateLimit-Remaining` and `X-RateLimit-Reset`; on `429 rate_limited` wait `Retry-After` seconds.

`Cache-Control` says per endpoint how long you may reuse an answer: valuations not at all, characteristics an hour, area statistics a day. In any case, keep Output no longer than 30 days, unless it belongs to an ongoing file about one home.
