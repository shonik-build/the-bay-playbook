# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

The Bay Playbook — a 17-page content site for Indian founders relocating to San Francisco. Sequenced across five stages (S0 deciding → S5 company). Each page is a self-contained HTML file rendered by a custom template DSL runtime (`support.js`) that loads React from unpkg.

**No build step.** Every `.dc.html` file is authored, shipped, and served as-is.

## Commands

- **Serve locally**: `python3 -m http.server 3000` from the repo root, then open `http://localhost:3000`.
- **Test a page**: Open the file directly in a browser. `file://` works.
- **Deploy**: `npx netlify-cli deploy --prod --dir=.` (or push to `main` if the GitHub → Netlify integration is set up).

## Architecture

Each `.dc.html` file is a **self-contained page** with this shape:

```html
<script src="./support.js"></script>   <!-- runtime -->
<x-dc>
  <helmet>…fonts + reset…</helmet>     <!-- fonts, body styles → head -->
  …template with {{ holes }}…
</x-dc>
<script type="text/x-dc" data-dc-script data-props="…">
  class Component extends DCLogic { … } <!-- the logic -->
</script>
```

`support.js` (~2000 lines) is the runtime: it parses `<x-dc>`, evaluates the class body, and calls React from unpkg to render. Do not modify or replace it.

Template DSL:
- `{{ path }}` — dotted lookup only. **No expressions.** Anything computed lives in `renderVals()`.
- `<sc-for list="{{ items }}" as="item">` — loop; `$index` in scope.
- `<sc-if value="{{ flag }}">` — conditional.
- Attributes: `x="{{ path }}"` = raw value; `x="a {{p}} b"` = string interpolation.
- Event handlers are camelCase (`onClick`, `onChange`).

Cross-page navigation uses **relative filenames**, not hashes. E.g. `href="Bay Playbook Apps.dc.html"`. Some pages (Guides, Directory) use in-page `#hash` for internal state.

## Design system

Read `HANDOFF.md` §3 for the canonical tokens. Do not change any value there without updating the file itself.

- **Fonts**: Archivo (structure, headings), IBM Plex Mono (labels/data), Caveat (pull quotes + wordmark), Silkscreen (the word "bay" in wordmark, nothing else).
- **Accent**: `--accent` CSS custom property, defaults to `#E8500A`. Each page sets it in `componentDidMount`.
- **Everything else** is inline `style=""` attributes. There are no CSS classes anywhere. Do not extract classes or add a stylesheet.

## Persisted state

Twelve `localStorage` keys hold user progress. Renaming any of these wipes user data:

```
bp-checklist                 landing master checklist
bp-getting-around-checks     Getting around
bp-remote-setup-checks       Remote setup
bp-packing                   Packing
bp-apps                      Apps (also gate toggles)
bp-emergency-card            Emergency (also blood group + contact)
bp-housing                   Housing
bp-ssn-dmv                   SSN / DMV (also visa selection)
bp-culture-checks            Culture
bp-networking                Networking (also 70-word DM draft)
bp-groceries                 Groceries
bp-weekend-checks            Weekends
bp-accelerators              Accelerators
```

## Four gotchas already burned in (from HANDOFF.md §8)

Do not re-introduce these:

1. **No inline `onError="{{ handler }}"` in the template.** Before hydration the browser evaluates the string as JS. Build the whole `<img>` in `renderVals()` and render it as `{{ p.logo }}`.
2. **`repeat(auto-fit, minmax(N,1fr))` collapses to one column** when `2×N + gap > available width`. The main content column is ~517px wide; grids use `minmax(240px,1fr)` in main and `minmax(220px,1fr)` inside dark panels.
3. **`white-space:nowrap` on a flex item overflows the card.** Flex items default to `min-width:auto`. Don't put `nowrap` on variable-length strings. Move to a mono eyebrow above the title if it won't fit beside it.
4. **Tooltips need a measured side-flip.** `max-width:calc(100vw - 44px)` doesn't help — it clamps against viewport width, not the trigger's distance from the edge. Measure `getBoundingClientRect()` at open time and drive `left`/`right` from state; set `box-sizing:border-box`.

## Route map

Every page has a route slug (matches the hash in the merged version described in HANDOFF §5):

- `home` → `index.html` / `Bay Playbook Landing v2.dc.html`
- `decisions` → 10-question index
- `directory` → filterable database
- `around`, `remote`, `packing`, `apps`, `emergency`, `housing`, `ssn`, `culture`, `networking`, `groceries`, `weekends`, `accel` → standalone pages
- `guides/<slug>` → inner router in `Bay Playbook Guides.dc.html` for 8 sub-slugs (visa, day07, sim, bank, money, healthcare, entity, short-term-housing)

Retired route redirects (still in `HANDOFF.md` §6):

```js
{ 'remote-setup': 'remote', 'getting-around': 'around' }
```

## Adding a page

1. Duplicate a similar `.dc.html` file (e.g. Emergency for a form-heavy page).
2. Change the `<helmet>` title, the template body, the class name, and the constants.
3. Add a cross-link from `index.html`'s sidebar `pageIndex` in the trailing `<script>` block.
4. Add the route to `HANDOFF.md` §5.
5. Test on `python3 -m http.server 3000`; verify no console errors and `document.documentElement.scrollWidth === clientWidth`.

## What NOT to do

- Do not add a build step, bundler, or framework wrapper.
- Do not add a stylesheet or extract classes — inline styles are the design intent.
- Do not swap out fonts. The four-family stack (Archivo/IBM Plex Mono/Caveat/Silkscreen) is load-bearing.
- Do not touch `support.js` unless you're rebuilding the runtime.
- Do not rename any `bp-*` localStorage key.
