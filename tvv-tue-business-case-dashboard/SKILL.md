---
name: tvv-tue-business-case-dashboard
description: 'Use when someone asks to create, build, write, or update a business case document, business case dashboard, or investment case for a TU/e initiative from source materials (PowerPoint decks, Word docs, PDFs). Also use when asked to turn a business case deck into an interactive dashboard, refresh a dashboard after the source deck changed, or price a tooling/vendor cost line that is not quantified in the source documents. Triggers on: "business case", "business case dashboard", "turn this deck into a dashboard", "update the dashboard from the deck".'
version: 1.0.0
category: process
scope: [tue]
triggers: [business case, business case dashboard, turn this deck into a dashboard, update the dashboard from the deck, investment case]
status: stable
created: '2026-07-31'
last_updated: '2026-07-31'
tags: [tue, business-case, dashboard, pptx, html]
platforms: [linux, macos, windows]
---

# tvv-tue-business-case-dashboard: TU/e Business Case Dashboard

Produces a self-contained, interactive HTML dashboard for a TU/e business case,
built from a folder of real source materials (PowerPoint decks, Word docs, PDFs,
emails) — never from assumed knowledge of the topic. Reuses one consistent TU/e
brand visual system across every business case so they read as one family of
documents.

For the extraction/verification scripts referenced below, see
[reference.md](reference.md). For the reusable HTML/CSS/JS skeleton, see
[template.html](template.html).

## Context

Needs: `python3` with `python-pptx` (and `python-docx` if Word docs are present),
`node` for validating the generated JS. Both are used via the Bash tool.

## Step-by-step workflow

**Step 1 — Inventory the source folder.** List every file in the target folder
(pptx, docx, pdf, msg, existing dashboards). Read any existing "evidence annex" or
similar traceability doc first if one exists — it tells you which source maps to
which claim and saves re-deriving that mapping from scratch.

**Step 2 — Pick the authoritative deck, but scan every pptx for hidden slides.**
Prefer the file with "Final" in its name, else the most recently modified pptx.
Run hidden-slide detection (reference.md §2) on **every** pptx in the folder, not
just the chosen one — you need to know what's hidden anywhere before you can be
sure a fact you find "confirmed" elsewhere isn't itself only sourced from a hidden
slide.

**Step 3 — Extract full content, tagged by visible/hidden.** Dump every slide's
text and tables (reference.md §3) to a temp file and Read it (not bash echo — see
the mojibake warning in reference.md §3). Do the same for any docx/pdf material
that's in scope.

**Step 4 — Treat hidden slides as deleted.** Do not use any fact, number,
scenario name, or framework that appears only on a hidden slide, even if it's more
complete or better-organized than the visible slides. If a decision slide is the
only visible source for the final ask, build the dashboard's option structure
around exactly what that slide shows (e.g. a flat "Do nothing / A / B" list), not
around a richer tiered framework that only exists on hidden slides. If you're not
sure whether something you want to use is hidden-slide-only, check — don't guess.

**Step 5 — Verify every number before using it.** If a figure looks broken or
placeholder-like (e.g. `€n–nK`), check the raw slide XML (reference.md §4) rather
than assuming it's an extraction glitch or inventing a plausible-looking number.
When a decision slide's headline total and a cost-detail slide's itemized
breakdown don't reconcile, don't force them to agree — surface both, cited to
their own slide, and note the discrepancy (reference.md §6).

**Step 6 — Build the dashboard from the template, not from scratch.** Check the
target folder for an existing TU/e dashboard HTML file first (e.g.
`dashboard-standalone.html`); if one exists, copy *its* `<style>` block instead of
`template.html`'s, so the new dashboard visually matches other dashboards already
in that project. Otherwise use `template.html` as-is. Populate the data object
per the contract documented in the template's script comment — do not redesign the
CSS or restructure the render logic for an individual business case.
If the deck's own decision slide presents only ONE recommended scenario (no named
peer options to switch between), don't force a tier-switcher onto it — drop the
switcher and render a single static scenario. A tier switch implies a comparison;
don't manufacture one the deck doesn't make. (If the deck is later revised to add
peer options back, add the switcher back then.)

**Step 7 — Every number and claim must be verifiable, but don't cite slides in
the output by default.** While building, trace every amount/note entry and every
benefit/risk bullet back to a specific visible slide or document in your own
head/notes — that discipline is what keeps you from inventing numbers. But unless
the user asks for an annotated/traceable version, do NOT write slide numbers
("slide 5", "Aannames, slide 14") into the dashboard's visible text or tooltips —
the dashboard should read as a self-contained business case document, not a
citation-heavy analysis of a deck. State the fact/rationale in plain prose
instead. Items that aren't yet costed or scoped go into a pending/pro-memoria line
with a plain-language reason, not into a fabricated number.

**Step 8 — If asked to price something the source docs don't cover** (e.g. a
tool's commercial licensing tier), research it with WebSearch/WebFetch per
reference.md §8: give a range, label it clearly as an external market estimate
(not a vendor quote), scale it to the business case's own numbers for
comparability, and keep it out of the priced KPIs — it's a pro-memoria/context
note, not part of the ask.

**Step 9 — Validate before delivering.** Extract the `<script>` block and run
`node --check` on it (reference.md §7). Re-parse the data object and confirm
category sums line up with the KPI headline figures, or that any mismatch is the
one already flagged from Step 5 — don't let a typo silently ship. Delete every
temp/dump/raw-xml/check-js file created during the session.

**Step 10 — Updating an existing dashboard.** If the user says the source deck
changed, follow reference.md §9: re-check hidden slides (they can change), re-dump
and diff mentally against the current data object, make targeted edits instead of
rewriting the file, re-validate, and summarize exactly what changed and which
slide justifies each change **in your chat reply** — not in the dashboard, which
stays citation-free per Step 7. If the deck's structure changed (e.g. dropped a
multi-option comparison down to one recommended scenario), rebuild that section's
structure too, not just the numbers inside it.

## Output format

- One self-contained `.html` file (inline CSS/JS, Google Fonts CDN link only
  external dependency), saved in the same folder as the source materials.
- Filename pattern: `<Project_Name>_Business_Case_Dashboard.html` (underscores,
  no spaces, matches the working folder's existing naming style if one exists).
- Content language matches the source documents (usually Dutch for TU/e internal
  decks) — don't translate unless asked.
- The dashboard's footer should name the source file(s) it was built from, in
  plain terms (e.g. "Bron: Apple Workplace Support business case, TU/e IT
  Services") — without slide numbers or hidden/visible commentary, unless the
  user has asked for a traceable/annotated version instead of a stand-alone one.

## Notes — what NOT to do

- Don't invent scenario names, tiers, or numbers that aren't traceable to a
  visible source. If the visible material is genuinely thin on a topic, say so in
  the dashboard (pro-memoria) rather than filling the gap with a plausible guess.
- Don't silently reconcile a source deck's own internal inconsistencies by
  averaging, rounding, or borrowing from a hidden slide — flag the inconsistency
  instead.
- Don't redesign the brand CSS per business case. If a case needs a visual
  component the template doesn't have, add it once, generally, and consider
  folding it back into `template.html` so the next business case gets it too.
- Don't fold external market-rate research into priced totals — it's supporting
  context for a decision, not a costed line, until someone gets a real quote.
- Don't leave temp extraction files (`_dump.txt`, `_*_raw.xml`, `_check.js`) behind
  in the user's project folder.
- Don't cite slide numbers in the dashboard's visible output by default (Step 7)
  and don't force a comparison UI onto a deck presenting a single recommendation
  (Step 6) — both were direct corrections from real use; see evals 3 and 4.
