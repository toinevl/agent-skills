---
name: tvv-escalate-blocked-fetch
description: Use whenever WebFetch, web_url_read, curl, or any raw HTTP fetch tool returns a 403, "bot detection", "access blocked", CAPTCHA, or similar block on a site whose content you actually need. A blocked raw fetch is a signal about that tool's fingerprint, not about whether the content is reachable — escalate to a real Playwright browser session on the exact same URL before concluding the site is unreachable or pivoting to a substitute source. Trigger whenever you catch yourself about to say "X is blocked/unreachable" or silently switch to an alternative source after a fetch error, without having tried a real browser first.
version: 1.0.0
category: meta
scope: [all]
triggers: ["403", "access blocked", "bot detection", "site unreachable", "cannot fetch", "CAPTCHA", "geo-restriction"]
status: stable
created: '2026-08-07'
last_updated: '2026-08-07'
---

# Escalate to a Real Browser When a Fetch Is Blocked

## Why This Matters

Real incident (2026-08-07): building a shopping list and later a custom
SearXNG engine, `web_url_read`/`WebFetch` against `ebay.nl` and
`developer.ebay.com` returned 403 ("bot detection"). The conclusion drawn
was "eBay isn't reachable through my tools" — and the response pivoted
straight to Marktplaats.nl as a substitute source, without ever trying the
Playwright browser tools that were sitting in the same tool list the whole
time.

The user had to explicitly say "en als je playwright gebruikt om die
pagina's te achterhalen" (what if you use Playwright to get those pages).
It worked on the **first try, every time** — eBay search results, eBay
listing prices, and eBay's own developer docs, all pages that a raw HTTP
fetch had just been blocked on seconds earlier.

**The root cause was not "eBay is blocked."** It was treating a block
returned by *one specific tool* (a raw, non-JS, non-browser HTTP client) as
equivalent to "this content cannot be retrieved at all" — instead of asking:
*is there a different tool in my kit whose fingerprint doesn't get flagged?*
A real browser session (real TLS/JS fingerprint, cookies, a genuine
User-Agent, can render JS-gated content) routinely gets through exactly the
kind of bot-detection that blocks `curl`-style fetches. This is the same
failure family as [[tvv-verify-external-target]] (treating one blocked
*source* as "nothing can be verified" instead of finding another *path* to
the same source) — here the missing step is another *tool*, not another
*URL*.

## The Rule

**A 403 / "bot detection" / CAPTCHA / access-blocked response from a raw
HTTP fetch tool is a signal about that tool's fingerprint — not proof the
content is unreachable.** Before declaring a site unreachable, or silently
pivoting to a substitute/alternative source, escalate to a real browser
(Playwright) on the exact same URL.

This does not override the `context-mode` guidance to prefer lightweight
fetches for efficiency — that guidance is about *cost* when content is
reachable. It says nothing about giving up when a fetch is blocked. Try the
cheap tool first; escalate the *tool*, not the *goal*, when it fails.

## Checklist: When a Fetch Returns Blocked

1. **Try the lightweight fetch first** (`WebFetch` / `web_url_read`). It's
   cheaper and keeps raw page bytes out of context — this remains the
   correct default, not something to skip.
2. **If it returns 403 / blocked / CAPTCHA / geo-restriction: do not
   conclude the site is unreachable and do not pivot to a different source
   yet.** Retry the *identical* URL via
   `mcp__plugin_playwright_playwright__browser_navigate`.
3. **Confirm you actually landed on real content**, not a CAPTCHA or consent
   wall — check the resulting page title/URL before trusting it.
4. **Extract data efficiently, not via `browser_snapshot` as a first
   move.** On a content-heavy page (search results, listings), a full
   accessibility snapshot can blow straight past the tool's max-token limit
   — confirmed in this session: `browser_snapshot` on an eBay.nl search
   results page errored at 200,531 characters. Instead use
   `browser_run_code_unsafe` with `page.evaluate(...)` to query the DOM
   directly and `return` only the structured fields actually needed (e.g.
   title/price/link as JSON). This keeps only derived data in context —
   the same "Think-in-Code" principle `context-mode` applies to other
   tools, applied here to Playwright.
5. **Only after the real browser session also fails** (same block, an
   unsolvable CAPTCHA, genuinely authenticated/paywalled content) is it time
   to conclude the source is unreachable — and even then, look for a public
   proxy source (per [[tvv-verify-external-target]]) before falling back to
   trained knowledge, and say explicitly what wasn't verified.
6. **Applies broadly**, not just to eBay: any site with bot-detection/WAF/
   Cloudflare-style protection — other marketplaces (Amazon, Bol.com,
   LinkedIn), ticketing sites, some SSO-gated corporate docs portals, etc.

## Bad Pattern (what happened)

```
Me:  *WebFetch ebay.nl/search → 403 "bot detection"*
Me:  "eBay blocks scraping, and there's no eBay engine in SearXNG either.
     Marktplaats.nl works though — let me use that instead."
Me:  *never tries Playwright, which was available the whole time*
User: "en als je playwright gebruikt om die pagina's te achterhalen"
Me:  *browser_navigate to the exact same URL → works immediately, every time*
```

## Good Pattern

```
Me:  *WebFetch ebay.nl/search → 403 "bot detection"*
Me:  "That's blocked for a raw fetch — let me try a real browser session
     on the same URL before assuming eBay itself is unreachable."
Me:  *browser_navigate → loads fine*
Me:  *browser_run_code_unsafe with page.evaluate() → extracts title/price
     as JSON, not a full snapshot*
Me:  "Confirmed: eBay is reachable, just not via raw HTTP. Here's the data."
```

## Quick Reference

| Situation | Don't | Do |
|---|---|---|
| `WebFetch`/`web_url_read` returns 403 / bot detection | Declare the site unreachable | Retry the same URL via Playwright `browser_navigate` |
| Real browser session loads fine | Assume the raw-fetch block meant "no data available" | Recognize it as a tool-fingerprint issue, not a content issue |
| Need structured data from a Playwright page | `browser_snapshot` on a listings/search page | `browser_run_code_unsafe` + `page.evaluate()` returning only the needed JSON fields |
| Browser session *also* blocked/CAPTCHA'd | Give up immediately | Look for a public proxy source before falling back to trained knowledge (state explicitly what's unverified) |
| Efficiency guidance says "prefer lightweight fetch" | Read that as "never use a browser tool" | Read it as "prefer lightweight when it works" — escalate the tool when it's blocked, not the goal |
