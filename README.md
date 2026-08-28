<p align="center">
  <img src="assets/logo.svg" alt="Nanobrowser Free AI" width="360" />
</p>

<h1 align="center">Nanobrowser Free AI</h1>

<p align="center">
  <b>A 100% free, private AI browser agent.</b><br/>
  Control your browser by typing plain-English tasks — powered by your own
  <b>free</b> Gemini or Groq API key. No credit card, no $200/month Operator bill.
</p>

<p align="center">
  <a href="https://chromewebstore.google.com/detail/nanobrowser/imbddededgmcgfhfpcjmijokokekbkal">Chrome Web Store</a> ·
  <a href="https://github.com/nanobrowser/nanobrowser">Upstream</a> ·
  <a href="#-quick-start">Quick Start</a> ·
  <a href="#-what-i-did-and-why">What I Did</a>
</p>

---

## What is this?

This is a **ready-to-use setup** of [Nanobrowser](https://github.com/nanobrowser/nanobrowser)
(an open-source AI web agent that lives in your browser) configured to run
**entirely on free LLM tiers**. The upstream project works great, but its docs
point you at paid models and leave you hitting confusing errors on free ones.

This repo bundles:
- ✅ A **complete setup guide** that actually works with free keys.
- ✅ A **patched build** of Nanobrowser that fixes the Groq `gpt-oss` tool-call
  error (so you can use Groq too, not just Gemini).
- ✅ Verified model names for 2026 (the old `gemini-2.0-flash` / `gemini-2.5-flash`
  are retired — use `gemini-3.6-flash`).

> Everything runs locally in your browser. Your API keys are stored only in your
> browser's local storage and are **never** sent to any third party.

---

## What I did and why

I wanted a free alternative to OpenAI Operator, so I cloned Nanobrowser and wired
it to free LLM providers. Along the way I hit (and fixed) real problems, all
verified with live API calls:

| Problem | What happened | Fix |
|---------|---------------|-----|
| **Groq `gpt-oss` tool-call error** | `Planning failed: ... attempted to call tool 'json' which was not in request.tools` (HTTP 400) | Nanobrowser forces a tool call that Groq's `gpt-oss` models reject. Patched `setWithStructuredOutput()` in `chrome-extension/src/background/agent/agents/base.ts` to use manual JSON extraction for `gpt-oss`. Built and confirmed working. |
| **Dead Gemini model names** | `gemini-2.0-flash` / `gemini-2.5-flash` return HTTP 404 in 2026 | Use **`gemini-3.6-flash`** (verified: returns a valid tool call, HTTP 200). |
| **OpenRouter `:free` not usable** | Free variants don't support tool calling (HTTP 404) | Don't use OpenRouter free tier for this; use Gemini or Groq. |
| **Wrong website guessed** | Agent opened `mytek.com` (a junk blog) instead of the real Tunisian store | Real URL is `https://www.mytek.tn`. |

**Result:** a working, free AI browser agent. Gemini works in the stock Web Store
build with zero code changes; Groq works via the patched build in this repo.

---

## Quick Start (Gemini — recommended, no coding)

1. **Install Nanobrowser** → https://chromewebstore.google.com/detail/nanobrowser/imbddededgmcgfhfpcjmijokokekbkal → "Add to Chrome".
2. **Get a free Gemini key** (1 min, no card) → https://aistudio.google.com/apikey
3. **Configure** (click the Nanobrowser icon → Settings gear):
   - Add Provider → **Gemini**, paste your key.
   - Planner → `gemini-3.6-flash`
   - Navigator → `gemini-3.6-flash`
   - Save.
4. **Type a task**, e.g.:
   > *"Open https://www.mytek.tn and search for PCs under 2000 Dts, then list the cheapest ones with prices."*

That's it. Gemini can also read screenshots, so it handles image-heavy or
French/Arabic pages well.

---

## Using Groq instead (needs the patched build)

Groq is faster for pure text but its `gpt-oss` models reject Nanobrowser's forced
tool calls — so you must use the **patched build from this repo**.

1. Get a free Groq key → https://console.groq.com/keys
2. Build the patched extension:

   ```bash
   cd path/to/this/repo
   npm install -g pnpm@9
   pnpm install --frozen-lockfile
   pnpm build          # output in ./dist
   ```

3. Load it unpacked: open `chrome://extensions/` → enable **Developer mode** →
   **Load unpacked** → select the `dist/` folder. Disable the Web Store copy if both are present.
4. Settings → Add Provider **Groq**, paste key, set:
   - Planner → `openai/gpt-oss-20b`
   - Navigator → `openai/gpt-oss-20b`
   - Save.

> Verified working Groq models (manual JSON extraction): `openai/gpt-oss-20b`,
> `qwen/qwen3.8-27b` (smartest), `allam-2-7b` (Arabic/French-native).
> `openai/gpt-oss-120b` is **unusable** on Groq (forces a tool call even with none defined).

---

## How to use it (writing good tasks)

In the Nanobrowser sidebar, type the instruction in plain English. **Do not paste
`curl` commands or `@url:` snippets** — just write the task:

- "Open https://www.mytek.tn and search for PCs under 2000 Dts, then list the cheapest ones with prices."
- "Go to amazon.com, search for 'usb-c cable', and list the 3 cheapest with prices."
- "Open my Gmail, find the latest email from my bank, and summarize it."

Watch the live Planner / Navigator / Validator status in the sidebar; ask
follow-ups after it finishes.

---

## FAQ

- **Is it really free?** Yes — Nanobrowser is open-source; the free tiers used here cost $0.
- **Is my data safe?** Yes — all runs locally; keys live in your browser only.
- **Gemini model 404?** You used an old name. Use `gemini-3.6-flash`.
- **Groq "tool 'json' not in request.tools"?** You're on the stock build with a `gpt-oss` model. Use Gemini, or build + load the patched `dist` here.
- **"No content extracted"?** You pasted a `curl`/URL snippet as the task. Type normal English.

---

## File map

```
assets/logo.svg          # repo logo
README.md                # this file
chrome-extension/...      # Nanobrowser source (patched: base.ts)
dist/                     # built extension (after pnpm build) for unpacked loading
packages/, pages/         # Nanobrowser source
```

## Credits

- Upstream: [nanobrowser/nanobrowser](https://github.com/nanobrowser/nanobrowser) (Apache-2.0)
- Free models: [Google Gemini](https://aistudio.google.com/apikey), [Groq](https://console.groq.com/keys)

---

<p align="center">Made for anyone who wants a free, private AI browser agent. ⭐ if it helped.</p>
