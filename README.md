<p align="center">
  <img src="assets/logo.svg" alt="Nanobrowser Free AI" width="360" />
</p>

<h1 align="center">Nanobrowser Free AI</h1>

<p align="center">
  <b>A 100% free, private AI browser agent.</b><br/>
  Control your browser by typing plain-English tasks — powered by your own
  <b>free</b> LLM API key. No credit card, no $200/month Operator bill.
</p>

<p align="center">
  <a href="https://chromewebstore.google.com/detail/nanobrowser/imbddededgmcgfhfpcjmijokokekbkal">Chrome Web Store</a> ·
  <a href="https://github.com/nanobrowser/nanobrowser">Upstream</a> ·
  <a href="#-quick-start">Quick Start</a> ·
  <a href="#-what-i-tried-and-what-actually-works">What Actually Works</a>
</p>

---

## What is this?

A **ready-to-use setup** of [Nanobrowser](https://github.com/nanobrowser/nanobrowser)
(an open-source AI web agent that lives in your browser) configured to run on
**free LLM tiers**. The upstream docs point you at paid models and leave you
hitting confusing errors on free ones. This repo bundles a complete, honest
setup guide based on real testing.

> Everything runs locally in your browser. Your API keys are stored only in your
> browser's local storage and are **never** sent to any third party.

---

## ⚠️ The short answer

**Use a free Gemini key.** It is the only free option that actually completes
browser tasks end-to-end. I tried Groq's `gpt-oss` models and they do **not** work
well in practice (details below) — don't waste time on them.

| Provider | Model | Actually works? | Notes |
|----------|-------|-----------------|-------|
| **Gemini** | `gemini-3.6-flash` | ✅ Yes | Free, fast, sees screenshots, supports tool-calling. **Use this.** |
| Groq | `openai/gpt-oss-20b` / `120b` | ❌ No (in practice) | Patch fixes the planner error, but models are too weak/slow to drive the Navigator reliably. |
| Groq | `qwen/qwen3.8-27b` | ⚠️ Planning only | Returns valid JSON, but still inferior to Gemini for real navigation. |
| OpenRouter `:free` | any | ❌ No | Free variants don't support tool calling (HTTP 404). |

---

## What I tried, and what actually works

I wanted a free OpenAI-Operator alternative, so I cloned Nanobrowser and wired it
to free providers. Here is what I verified with **live API calls** — and the
honest outcome:

1. **Groq `gpt-oss` tool-call error (real bug, partial fix).**
   Nanobrowser forces the LLM to return its plan via a *forced tool call* named
   `json`. Groq's `gpt-oss` models reject forced tool calls →
   `attempted to call tool 'json' which was not in request.tools` (HTTP 400).
   I patched `setWithStructuredOutput()` in
   `chrome-extension/src/background/agent/agents/base.ts` to use manual JSON
   extraction for `gpt-oss`, which **fixed the planner error** (the model then
   returns valid plan JSON).

   **But:** even with the patch, `gpt-oss-120b` is too slow and `gpt-oss-20b` is
   too weak to actually complete browser tasks — the Navigator produces wrong or
   malformed actions, and latency is poor. So **Groq gpt-oss is not usable for
   real automation.** The patch is kept here for reference, but Gemini is the
   practical choice.

2. **Use `gemini-3.6-flash`** (verified: returns a valid tool call, HTTP 200, and completes tasks).

3. **OpenRouter `:free` not usable.** Free variants don't support tool calling
   (HTTP 404). Skip it.

4. **Wrong website guessed.** The agent opened `mytek.com` (a junk blog) instead
   of the real Tunisian store `https://www.mytek.tn`. Always give the full URL.

---

## Quick Start (Gemini — the one that works)

1. **Install Nanobrowser** → https://chromewebstore.google.com/detail/nanobrowser/imbddededgmcgfhfpcjmijokokekbkal → "Add to Chrome".
2. **Get a free Gemini key** (1 min, no card) → https://aistudio.google.com/apikey
3. **Configure** (click the Nanobrowser icon → Settings gear):
   - Add Provider → **Gemini**, paste your key.
   - Planner → `gemini-3.6-flash`
   - Navigator → `gemini-3.6-flash`
   - Save.
4. **Type a task**, e.g.:
   > *"Open https://www.mytek.tn and search for PCs under 2000 Dts, then list the cheapest ones with prices."*

Gemini reads screenshots too, so it handles image-heavy or French/Arabic pages well.

---

## Using Groq (experimental — not recommended)

Groq is faster for raw text, but its free `gpt-oss` models **do not complete
browser tasks reliably** (see above). If you still want to try:

1. Free Groq key → https://console.groq.com/keys
2. Build the patched extension:

   ```bash
   cd path/to/this/repo
   npm install -g pnpm@9
   pnpm install --frozen-lockfile
   pnpm build            # output in ./dist
   ```

3. Load unpacked: `chrome://extensions/` → Developer mode → Load unpacked → `dist/`.
   Disable the Web Store copy if both are present.
4. Settings → Add Provider **Groq**, paste key, set:
   - Planner → `openai/gpt-oss-20b` (or `qwen/qwen3.8-27b`)
   - Navigator → `openai/gpt-oss-20b`
   - Save.

> Expect the agent to plan but fail or stall during navigation. For working
> results, use Gemini.

---

## How to use it (writing good tasks)

In the Nanobrowser sidebar, type the instruction in plain English. **Do not paste
`curl` commands or `@url:` snippets** — just write the task, and give the full URL:

- "Open https://www.mytek.tn and search for PCs under 2000 Dts, then list the cheapest ones with prices."
- "Go to amazon.com, search for 'usb-c cable', and list the 3 cheapest with prices."

Watch the live Planner / Navigator / Validator status; ask follow-ups after it finishes.

---

## FAQ

- **Is it really free?** Yes — Nanobrowser is open-source; Gemini's free tier costs $0.
- **Is my data safe?** Yes — all runs locally; keys live in your browser only.
- **Which free model should I use?** **Gemini `gemini-3.6-flash`.** Groq gpt-oss does not work for real tasks.
- **Gemini model 404?** You used an old name. Use `gemini-3.6-flash`.
- **Groq "tool 'json' not in request.tools"?** gpt-oss rejects forced tool calls. The patch fixes the planner error, but Groq still isn't reliable for navigation — switch to Gemini.
- **"No content extracted"?** You pasted a `curl`/URL snippet as the task. Type normal English.

---

## File map

```
assets/logo.svg          # repo logo
README.md                # this file
chrome-extension/...      # Nanobrowser source (patched: base.ts, Groq gpt-oss fix — reference only)
dist/                     # built extension (after pnpm build) for unpacked loading
packages/, pages/         # Nanobrowser source
```

## Credits

- Upstream: [nanobrowser/nanobrowser](https://github.com/nanobrowser/nanobrowser) (Apache-2.0)
- Free model: [Google Gemini](https://aistudio.google.com/apikey)

---

<p align="center">Made for anyone who wants a free, private AI browser agent. ⭐ if it helped.</p>
