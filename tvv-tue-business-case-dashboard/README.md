# tvv-tue-business-case-dashboard

Builds a self-contained, interactive HTML dashboard for a TU/e business case
from a folder of real source materials (PowerPoint decks, Word docs, PDFs) —
reusing one consistent TU/e brand visual system so every business case reads as
part of the same family of documents.

## Why this exists

Business case decks accumulate hidden slides (superseded scenarios, dropped
numbers, old drafts kept as speaker backup) and internal inconsistencies
(a decision slide's headline figure drifting from a cost-detail slide's
itemized build-up). Turning one into a dashboard by hand means re-deriving the
same discipline every time: which slides are actually authoritative, which
numbers are placeholders versus real, and how to present a discrepancy
honestly instead of picking one number and hoping nobody notices.

This skill encodes that discipline as a repeatable workflow, plus a reusable
HTML/CSS/JS template so the visual language doesn't get reinvented per case.

## What it produces

One `.html` file, self-contained except for a Google Fonts CDN link, with:

- A current-situation summary and key stats
- Either a tier-switcher (if the deck presents named options to compare) or a
  single static scenario (if it presents one recommendation) — never a
  switcher for a comparison the deck doesn't actually make
- An interactive cost breakdown chart with hover tooltips
- A pending/open-items panel for anything not yet costed or scoped
- A plain-language decision ask

By default the dashboard reads as a stand-alone document — no slide-number
citations in its visible text. Traceability lives in the building process
(every fact is checked against a specific visible slide before use) and in
the assistant's chat summary when updating an existing dashboard, not in the
artifact itself.

## Hard rule: hidden slides are deleted

PowerPoint's "Hide Slide" flag (`show="0"` in the slide XML) is checked on
every pptx in the folder, not just the one named as authoritative. Anything
that appears only on a hidden slide — a scenario name, a number, a whole
framework — is treated as if it doesn't exist, even if it's more detailed
than what's visible, and even if the same content shows up unhidden in an
older draft file sitting in the same folder.

## See also

- [SKILL.md](SKILL.md) — the full step-by-step workflow
- [reference.md](reference.md) — extraction/verification scripts (hidden-slide
  detection, mojibake-vs-real-placeholder diagnosis, reconciliation checks)
- [template.html](template.html) — the reusable dashboard skeleton
