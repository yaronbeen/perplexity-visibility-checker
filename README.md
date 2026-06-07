# 🔮 perplexity-visibility-checker

**See whether Perplexity cites your brand — live.**

Perplexity answers every question with a ranked list of **sources**. That list
*is* the game: if your domain isn't on it, you don't exist at the moment your
buyer is deciding.

Type your brand and a buyer question. This tool runs a **real, live Perplexity
session** (via Bright Data's Perplexity scraper) and tells you whether you were:

- **Cited** as a source (and at what rank),
- **Named** in the answer text, or
- **Invisible** — with the exact ranked sources Perplexity trusts instead.

> Part of a 3-tool series on AI brand visibility. See also the
> [ChatGPT checker](https://github.com/yaronbeen/chatgpt-visibility-checker)
> and the combined [invisible-to-ai](https://github.com/yaronbeen/invisible-to-ai).
>
> Inspired by *["My SaaS Was Invisible to ChatGPT"](https://medium.com/@yaron.been/my-saas-was-invisible-to-chatgpt-i-built-a-scraping-pipeline-to-fix-it-4703bbed2345)*.

---

## How it works

```
Browser ──► Cloudflare Worker (same-origin proxy) ──► Bright Data Perplexity scraper
   ▲                                                          │
   └──────────  answer + ranked sources + verdict  ◄──────────┘
```

1. You enter a **brand**, a **buyer question**, a **location**, and **your own
   Bright Data token**.
2. The Worker triggers the Perplexity scraper (`gd_m7dhdot1vw9a7gc1n`) and returns
   a snapshot ID.
3. The page polls until the run is `ready` (~30s), then renders the answer, the
   full ranked citation list, "people also ask", and a verdict.
4. Export as Markdown, JSON, or CSV.

| Engine | Dataset ID | Key inputs |
| --- | --- | --- |
| Perplexity | `gd_m7dhdot1vw9a7gc1n` | `url, prompt, country, index` |

Unlike ChatGPT, Perplexity **always** returns a ranked source list — which makes
it the cleanest signal of all for AI search visibility.

---

## Bring Your Own Key (BYOK) — zero secrets

**There is no API token in this repo or in the deployed Worker.** Each visitor
pastes their **own** Bright Data token; it's forwarded per request and never
stored, logged, or persisted. (A proxy is needed only because Bright Data's API
doesn't send CORS headers, so a browser can't call it directly.)

Get a free token at [brightdata.com](https://brightdata.com) → *Settings → API keys*.

---

## Run it yourself

```bash
npm install
npm run dev        # http://localhost:8787
npm run deploy     # deploy to your Cloudflare account
```

No secrets to configure. Click **"See a real sample"** to view a real result for
*ROASPIG* without using a token.

### Raw API example (synchronous)

```bash
curl -H "Authorization: Bearer $BRIGHT_DATA_API_TOKEN" -H "Content-Type: application/json" \
  -d '{"input":[{"url":"https://www.perplexity.ai","prompt":"best AI ad creative tools in 2026","country":"US","index":1}]}' \
  "https://api.brightdata.com/datasets/v3/scrape?dataset_id=gd_m7dhdot1vw9a7gc1n&notify=false&include_errors=true"
```

(For long jobs use the async pattern: `POST /datasets/v3/trigger` → poll
`GET /datasets/v3/progress/{id}` → download `GET /datasets/v3/snapshot/{id}?format=json`.)

---

## Structure

```
perplexity-visibility-checker/
├── public/
│   ├── index.html     # the whole front-end (neo-brutalist, vanilla JS)
│   └── sample.json    # a real sample result for the no-key demo
├── src/
│   └── worker.js      # stateless BYOK proxy
├── wrangler.jsonc
└── package.json
```

---

Built with love by **[Yaron · nofluff.online](https://nofluff.online)** · powered by **Bright Data**. MIT licensed.
