# Reference: extraction & verification scripts

All snippets assume `python3` with `python-pptx` (and `python-docx` for Word docs)
installed, and `node` available for JS validation. Run via the Bash tool.

## 1. Enumerate source files and pick the authoritative one

```bash
ls "<folder>"
```

Prefer, in this order: a file with "Final" in the name > the most recently modified
pptx > a "v2"/highest-version-number file. If genuinely ambiguous, ask the user which
deck is authoritative rather than guessing — don't silently pick one and proceed.

If a pptx read fails with `PackageNotFoundError` / `Permission denied`, the file is
almost certainly open in PowerPoint right now. Tell the user, don't retry in a loop.

## 2. Detect hidden slides — do this for EVERY pptx in the folder, every time

```python
from pptx import Presentation
p = Presentation('<file>.pptx')
for i, slide in enumerate(p.slides):
    show = slide._element.get('show')
    print(i+1, 'HIDDEN' if show == '0' else 'visible')
```

A slide is hidden iff `show="0"` on its `<p:sld>` element. This is a real PowerPoint
"Hide Slide" flag, not a heuristic. **Never build any part of the dashboard's
structure or numbers from content that appears ONLY on a hidden slide** — treat it
as if it were deleted, even if it's more detailed or more "complete" than what's
visible. If the same fact independently appears on a visible slide (in the same
deck or a different one — e.g. an earlier draft deck), it's fair game; cite that
visible source, not the hidden one.

When a decision/summary slide keeps a label or number that was clearly authored
against a now-hidden slide (e.g. a scenario name like "Typical" surviving after the
slide defining Lean/Typical/Complex got hidden), don't silently launder it back into
a full framework — use the plain number/ask as presented, drop the orphaned label,
and rebuild the option structure from what the visible decision slide actually shows
(often just a flat list of options, not a tiered comparison).

## 3. Full text + table extraction, tagged with hidden/visible

```python
from pptx import Presentation
p = Presentation('<file>.pptx')
with open('_dump.txt', 'w', encoding='utf-8') as out:
    for i, slide in enumerate(p.slides):
        hidden = slide._element.get('show') == '0'
        out.write(f'=== SLIDE {i+1} {"HIDDEN" if hidden else "VISIBLE"} ===\n')
        for shape in slide.shapes:
            if shape.has_text_frame:
                for para in shape.text_frame.paragraphs:
                    text = ''.join(run.text for run in para.runs)
                    if text.strip():
                        out.write(text + '\n')
            if shape.has_table:
                out.write('[TABLE]\n')
                for row in shape.table.rows:
                    out.write(' | '.join(cell.text for cell in row.cells) + '\n')
        out.write('\n')
```

Then Read the dump file with the Read tool (not `cat`/bash echo) — bash's console
codepage can mangle non-ASCII characters (€, –, é) into `�`, which looks exactly
like real data corruption but usually isn't. Read renders UTF-8 correctly.

## 4. Distinguish "corrupted in my extraction" from "actually broken in the deck"

If a headline number looks garbled (e.g. `€n–nK/jaar`), check the raw XML before
concluding anything:

```python
import re
xml = open('_slide_raw.xml', encoding='utf-8').read()  # from zipfile.read('ppt/slides/slideN.xml')
runs = re.findall(r'<a:t>(.*?)</a:t>', xml, re.S)
for i, r in enumerate(runs): print(i, repr(r))
```

If the literal run text contains ASCII `n` characters where digits should be (not a
`�` replacement character), the author left an unfilled placeholder in the deck
itself — it's a real authoring defect, not an extraction bug. In that case, derive
the real number from the slide's own sub-bullets/components instead of guessing a
digit, and don't repeat the placeholder anywhere in the dashboard.

## 5. Docx extraction (paragraphs + tables)

```python
import docx
d = docx.Document('<file>.docx')
for p in d.paragraphs:
    print(p.text)
for t in d.tables:
    for row in t.rows:
        print(' | '.join(c.text for c in row.cells))
```

## 6. Reconcile headline numbers against component breakdowns

Business-case decks routinely have a "decision slide" headline figure and a
separate "cost detail slide" itemized build-up. They often don't sum to exactly the
same total (rounding, later edits to one slide but not the other). When they
disagree:
- **Do not** silently pick one, average them, or quietly borrow a number from a
  hidden slide to force a reconciliation.
- **Do** use the decision slide's number as the KPI headline (it's literally what's
  being asked for approval), show the detail slide's own breakdown in the chart/
  tooltips, and if the gap is material, say so in a footnote so the inconsistency in
  the source is visible rather than papered over.
- After building the `B.lines`/`B.cat_totals` data object, verify arithmetic with a
  small Node check before delivering:

```bash
node -e "
const fs = require('fs');
const content = fs.readFileSync('<dashboard>.html','utf-8');
const m = content.match(/const B = (\{[\s\S]*?\});\nconst cap/);
const B = eval('(' + m[1] + ')');
B.tiers.forEach(t => {
  const sum = B.categories.reduce((s,c)=> typeof B.cat_totals[t][c]==='number' ? s+B.cat_totals[t][c] : s, 0);
  console.log(t, JSON.stringify(B.cat_totals[t]), 'sum=', sum, 'vs KPI', B.kpi[t].structural, B.kpi[t].oneoff);
});
"
```

## 7. Validate the JS before delivering

```bash
python3 -c "
import re
content = open('<dashboard>.html', encoding='utf-8').read()
m = re.search(r'<script>(.*)</script>', content, re.S)
open('_check.js','w',encoding='utf-8').write(m.group(1))
"
node --check _check.js && echo OK
rm -f _check.js
```

Always clean up every `_dump.txt`, `_*_raw.xml`, and `_check.js` file created during
the session before finishing — they're working scratch, not deliverables.

## 8. Researching a cost line the source docs don't cover

If asked to price something the internal decks don't quantify (e.g. "what would the
paid tier of tool X cost"), use WebSearch/WebFetch, then:
- Give a range, not a false-precision point estimate, and say plainly that it's a
  third-party/market estimate, not a vendor quote or an official TU/e price.
- Scale it to the actual numbers already in the business case (device counts, node
  counts, FTE) so it's comparable, but keep the illustrative scaling clearly labeled
  as illustrative.
- Never fold an external estimate into the dashboard's priced totals/KPIs — it goes
  in a pro-memoria line or a clearly separate note, exactly like any other
  not-yet-committed cost.
- Cite sources (WebSearch requires this anyway).

## 9. Updating a dashboard after the source deck changes

When the user says the deck was updated:
1. Re-run hidden-slide detection (step 2) — hidden/visible status can change.
2. Re-dump all slides (step 3) and diff mentally against what's already in the
   dashboard's `B` object — most updates are a handful of number/wording changes,
   not a full rebuild.
3. Use targeted Edit calls on the existing HTML file rather than rewriting it
   wholesale, so unrelated content and prior review don't get discarded.
4. Re-run the reconciliation check (step 6) and JS validation (step 7) after editing.
5. Summarize exactly what changed and why, citing the slide **in your chat reply**
   to the user — the same way you would in a PR description, since they're
   tracking a moving source of truth and need to know what moved. This is
   different from the dashboard's own visible text, which should stay
   citation-free by default (SKILL.md Step 7) — the traceability lives in your
   summary and your own verification process, not in the artifact.
6. If a targeted edit turns out not to be enough — e.g. the deck removed a
   multi-option comparison down to a single recommended scenario, or added one —
   the dashboard's *structure* (tier-switcher vs. single-scenario layout) needs
   to change too, not just the numbers inside it. Don't patch numbers into a
   structure the deck no longer supports; rebuild that section properly.
