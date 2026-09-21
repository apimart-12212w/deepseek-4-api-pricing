# DeepSeek 4 API Pricing (deepseek-v4-pro / deepseek-v4-flash)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

> **$1.03 / $3.09 per million tokens** (input / output, effective) — one OpenAI-compatible endpoint at `https://api.apimart.ai/v1`, no monthly plan required. *(observed 2026-09-17)*

**[Get an API key](https://go.apimart.ai/k-403824)** · **[Live pricing](https://go.apimart.ai/k-080a52)** · **[Model page](https://go.apimart.ai/k-61c7e4)** · [⚡ 60-second quickstart](#quickstart)

**Why teams call DeepSeek 4 Pro (`deepseek-v4-pro`) through APIMart**

- **One key, entire catalog.** The same `https://api.apimart.ai/v1` base URL and `Authorization` header reach DeepSeek 4 Pro (`deepseek-v4-pro`) and 300+ other image, video and language models — switch the `model` field, not your client.
- **$1 minimum, pay as you go.** No subscription and no prepaid plan to size up front: top up from $1 and spend it on calls. There is no free quota to burn through first, so the price in this table is the price you pay.
- **The charge comes back in the response.** Every call reports the amount billed (`cost` / `credits_cost`), so a spend number is read per call instead of guessed at month end.
- **Drop-in OpenAI shape.** `POST /v1/chat/completions` with the same request body your client already sends; only `base_url` and `model` change.

<!-- /conv-kit:v1 -->

What DeepSeek 4 costs per million tokens on each tier, with worked examples: fresh input, cached input, output, and the crossover point where the cheaper `-flash` tier stops being the right answer.

## Model ids

| Model id | Tier | Typical use |
| --- | --- | --- |
| `deepseek-v4-pro` | highest quality | hard prompts, long-form reasoning |
| `deepseek-v4-flash` | fast/cheap tier | high-volume extraction, classification, routing |

Endpoint: `POST https://api.apimart.ai/v1/chat/completions` (OpenAI-compatible). **Streaming is the default** — pass
`stream: false` when you want one JSON object back.

## Pricing (per million tokens)

<!-- pricing:token:start -->
| Token direction | List price / 1M | Effective price / 1M |
| --- | --- | --- |
| **deepseek-v4-flash** | | |
| cached_input | $0.0857 | $0.0686 |
| input | $0.4286 | $0.3429 |
| output | $1.29 | $1.03 |
| **deepseek-v4-pro** | | |
| cached_input | $0.2571 | $0.2057 |
| input | $1.29 | $1.03 |
| output | $3.86 | $3.09 |
<!-- pricing:token:end -->

The effective column is what you pay after the default group discount; [`data/model.json`](data/model.json) is refreshed
daily by CI, and the `usage` block in every response tells you exactly which tokens were billed.

## Verified capabilities

| Capability | Verified behaviour |
| --- | --- |
| Per-million-token billing | every response returns `usage` with prompt, cached and completion token counts |
| Cache discount | cached input is billed at a fraction of fresh input on both tiers |
| Tier spread | `-flash` costs roughly a third of `-pro` per million tokens in every direction |
| Worked examples | the table below prices the real calls recorded in `data/samples.json` |

## Quickstart

```bash
curl -sS https://api.apimart.ai/v1/chat/completions \
  -H "Authorization: Bearer $APIMART_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"deepseek-v4-pro","stream":false,"messages":[{"role":"user","content":"Name three retry rules."}]}'
```

```python
import os, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

payload = requests.post(f"{BASE}/chat/completions", headers=HEADERS, timeout=120, json={
    "model": "deepseek-v4-pro", "stream": False,
    "messages": [{"role": "user", "content": "Name three retry rules."}],
}).json()
print(payload["choices"][0]["message"]["content"])
print(payload["usage"])
```

Streaming, tool calling and JSON mode examples are in [`examples/`](examples) (`t_curl.sh`, `python_chat.py`).

## Real call outputs

These rows are actual completions recorded from this route, with the token usage the API returned and the cost computed
from the effective rates above.

| Prompt | Response excerpt | Tokens (in/out) | Reported cost |
| --- | --- | --- | --- |
| `Explain when cached input pricing beats a lower output price, with a one-line formula.` | Cached input pricing means you pay a premium for a pre‑computed, instantly available piece of data (the cached input) instead of paying a lower price for a freshly generated result (the output). The higher price is worth… | 21 / 2793 | $0.0086 |
| `Write a 15-line Python function that routes a batch job to the cheapest model from a dict ` | ```python def route_batch(model_prices: dict[str, tuple[float, float]], input_tokens: int, output_tokens: int) -> str:     """     Routes a batch job to the cheapest model.          Args:         model_prices: dict mappi… | 41 / 1161 | $0.0036 |

Full transcripts (including longer answers) are in [`data/samples.json`](data/samples.json).

<!-- conv-kit:v1:fix -->
## First-call troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `401` / `invalid api key` | key missing, truncated, or a stray newline pasted into the header | Re-copy it from the console; the header is `Authorization: Bearer $APIMART_API_KEY` |
| balance / credit error | the account has no balance | Top up from $1 in the console — there is no free quota to fall back on |
| `429` | concurrent requests on one key | Back off, then retry the same request with the same `Idempotency-Key` |
| `400` / model not found | wrong route for the id: the per-unit alias needs its `version`, the official id must not send one | Copy the exact `model` value from the route table above |
| task ends `failed` | prompt rejected by the filter, or a reference image URL expired | Re-submit with a **new** `Idempotency-Key` and re-host the reference image |
| result URL stops working | result links expire | Download the file as soon as the task reports `completed` |
<!-- /conv-kit:v1:fix -->

## FAQ

**How much does DeepSeek 4 cost per million tokens?**

The table above lists list and effective rates for input, cached input and output on both tiers. Effective rates include the default group discount.

**When does `-pro` pay for itself?**

When the cheaper tier fails your acceptance checks and you have to re-run. Two failed flash attempts usually cost more than one pro call, so measure rework rate rather than unit price.

**Does caching change the arithmetic?**

Yes, materially: repeated prefixes bill at the cached-input rate, which is the single biggest lever for long-context workloads. The `usage` block reports the cached token count so you can verify it.

**How do I reconcile spend against an invoice?**

Sum the token counts from every response and apply the effective rates; token-billed routes do not report a per-request `cost` field the way per-image routes do.

## Related searches

- `deepseek 4 api pricing`
- `deepseek api key`
- `llm api pricing comparison`
- `cheapest llm api`
- `cached input pricing`
- `llm api cost comparison`
- `openai compatible api`

<!-- conv-kit:v1:cta -->
---

**Start with $1.** [Get an API key](https://go.apimart.ai/k-403824) → [check live pricing](https://go.apimart.ai/k-080a52) → [open DeepSeek 4 Pro (`deepseek-v4-pro`) in the model library](https://go.apimart.ai/k-61c7e4). The first call is three steps: submit, poll `task_id`, read the charged amount off the response.
<!-- /conv-kit:v1:cta -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Browse the model catalog | <https://go.apimart.ai/k-61c7e4> | `apimart.ai/model` |
| Current pricing page | <https://go.apimart.ai/k-080a52> | `apimart.ai/pricing` |
| Get an API key | <https://go.apimart.ai/k-403824> | `apimart.ai/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

DeepSeek 4 is a third-party model served through APIMart; this repository publishes model ids, measured prices and
real call outputs, and does not claim official status. Model names, prices and documentation belong to their respective
owners. Endpoint reference: [https://docs.apimart.ai/en/api-reference/texts/general/chat-completions](https://docs.apimart.ai/en/api-reference/texts/general/chat-completions).

## Repository map

```text
README.md             model ids, token pricing, verified capabilities, real outputs
data/model.json       token rates for every tier (CI-refreshed)
data/samples.json     recorded completions with usage and computed cost
tools/snapshot.py     refresh pricing from the public payload
tools/check_links.py  attribution guard
examples/             curl and Python clients (streaming, tools, JSON mode)
.github/workflows/    daily price refresh + validation
```

## License

MIT — see [LICENSE](LICENSE).
