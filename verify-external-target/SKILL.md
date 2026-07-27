---
name: verify-external-target
description: Use BEFORE implementing any task that asks you to align, match, mirror, or replicate an external brand, product, competitor's UI, or published specification. Fetch and verify real values (colors, fonts, endpoints, schemas, copy) from the actual live/public source before writing code — never substitute trained/memorized knowledge for a fact that could be checked. If the ideal source (an internal doc, an auth-walled portal) is unreachable, look for the next-best PUBLIC proxy (the org's own live website, published docs, a public repo) before ever falling back to approximation. Trigger whenever you catch yourself about to write "approximately", "typically", "usually looks like", or a specific hex/name/value you recall but haven't just fetched.
---

# Verify External Target Before Implementing

## Why This Matters

Real incident: asked to rebrand a frontend to match TU/e (Eindhoven University
of Technology)'s visual identity. The user pointed to TU/e's internal
corporate-identity SharePoint page; it returned 403 (SSO-gated, unreachable).
Rather than looking for another way to verify, the fallback was "use TU/e's
well-known public identity from memory" — a guessed hex code (`#e4002b`), a
guessed font (`Inter`), and guessed rounded/shadowed card styling. All of it
was shipped to a real deployment (a sandbox environment) and presented as a
TU/e-aligned rebrand.

It wasn't. The user looked at the deployed site and said: *"I don't see any
use of the TU/e branding beyond the use of red text in some places. Please
verify looking at tue.nl and subpages."*

Fetching tue.nl's actual production CSS took a handful of `curl` calls and
found: the real brand red is `#c72125` (not `#e4002b`), the real body font is
`Lato` (not `Inter`), and the site is almost entirely **sharp-cornered with no
shadows** (`border-radius:0` dominates their CSS) — the opposite of what had
just been guessed and deployed. Every one of these was independently
verifiable, minutes of work, and freely available with no auth wall — the
whole time.

**The root cause was not "the official source was blocked."** It was
treating "the one privileged source is blocked" as equivalent to "nothing can
be verified," instead of asking: *is there a public proxy for this same
ground truth?* An organization's own live website is almost always a better,
more current, and more specific source than trained knowledge about "what
universities typically look like" — and it's usually one `curl`/WebFetch away.

## The Rule

**Before implementing anything that must match an external target, verify it
against the target's actual live artifact — don't approximate from memory,
even when your first-choice source is unreachable.**

This applies to any external-alignment task, not just visual branding:
- Brand identity (colors, fonts, logo, spacing, tone) → fetch the org's live
  website CSS/HTML, not "what's typical for this kind of org"
- API integration ("match their contract") → fetch their published OpenAPI
  spec / docs page, not "REST APIs usually look like"
- Competitor feature parity ("build something like X") → look at X's actual
  live product, not a memory of what it does
- Following a spec/RFC → fetch the actual spec text, not a paraphrase from
  training

## Checklist: Before Writing Any Code

1. **Identify every fact you're about to assert that originates outside this
   repo** — a hex code, a font name, an endpoint shape, a copy string, a
   layout convention. Each one is a claim that needs a source.
2. **Try the source the user gave you first.** If it's reachable, fetch it
   (WebFetch, curl, `gh api`, whatever fits). Extract exact values, don't
   summarize-and-forget them — quote the real hex codes, real font-family
   declarations, real endpoint paths.
3. **If that source is blocked (403, SSO wall, private), do NOT jump to
   approximation.** Ask instead: *does this same organization/product have a
   public artifact that carries the same ground truth?*
   - A company's internal brand PDF is gated → their public **website**
     almost certainly isn't.
   - A private API spec is gated → the product's **public docs page** or a
     **published Postman/OpenAPI file** often isn't.
   - An internal design system repo is gated → their **public-facing app**
     still ships the real CSS in the browser.
4. **Fetch the public proxy directly — don't ask an AI summarizer to describe
   it from a screenshot alone.** For CSS/structured data, `curl` the actual
   stylesheet/HTML and `grep` for the real values (font-family, hex colors,
   border-radius, etc.) — a rendered screenshot confirms the *character* of a
   design, but only the raw source gives you copy-able precise values.
   Combine both: raw source for exact values, screenshot for gestalt/layout
   confirmation.
5. **Only if truly nothing is fetchable** (no public proxy exists at all) —
   fall back to approximation, but:
   - Say so explicitly, per value, not as one blanket caveat
   - Get the user's explicit confirmation before implementing broadly
   - Isolate every guessed value behind a single named token/variable so
     correcting it later is a one-line change, not a re-do
6. **In plan mode:** when the task is "align to / match / mirror external
   target X," add "fetch and inspect X's live artifact" as an explicit
   Phase-1 research step — same weight as exploring the internal repo. Don't
   let repo exploration alone stand in for external verification.

## Bad Pattern (what happened)

```
User: "align the app to TU/e branding"
Me:  *fetches SharePoint link → 403*
Me:  "I'll use TU/e's well-known public identity as best-effort" (from memory)
Me:  *ships guessed #e4002b, guessed Inter font, guessed rounded cards*
User: "I don't see TU/e branding here beyond some red text. Verify against tue.nl."
Me:  *finally curls tue.nl's real CSS* → finds real values differ on every axis
Me:  *redoes the whole rebrand, redeploys*
```

Two full implementation+deploy cycles for a fact-check that should have
happened before the first line of CSS was written.

## Good Pattern

```
User: "align the app to TU/e branding"
Me:  *fetches SharePoint link → 403*
Me:  "That's blocked — but tue.nl itself ships their real CSS. Let me check that."
Me:  *curl tue.nl, curl their stylesheet, grep for :root variables, font-family, border-radius*
Me:  "Verified: brand red is #c72125, body font is Lato, corners are sharp (radius:0)."
Me:  *implements once, correctly, first time*
```

## Quick Reference

| Situation | Don't | Do |
|---|---|---|
| Official doc is auth-walled | Approximate from memory | Check if the org has a public site/docs carrying the same info |
| You "recall" a brand color | Type the hex from memory | `curl`/WebFetch the live site's CSS and grep for the real value |
| Screenshot shows a design | Describe its "vibe" and guess values | Pull the raw HTML/CSS for exact values; use the screenshot only to confirm layout/gestalt |
| Task is "match/mirror/align to X" | Treat repo exploration as sufficient research | Add "fetch X's live artifact" as its own required research step |
| Best-effort fallback is truly unavoidable | One blanket "not verified" caveat | Per-value caveat + user confirmation + single-token isolation for easy correction later |
