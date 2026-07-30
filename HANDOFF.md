# The Bay Playbook — build handoff

You have 17 files and this document. Your job: merge them into **one self-contained HTML file** that works offline, with hash-based routing between all 20 guides, preserving the existing design exactly.

Do not redesign anything. Every page in the zip is finished and reviewed. This is an assembly job.

---

## 1. What's in the zip

| File | Route | Notes |
|---|---|---|
| `index.dc.html` | `#home` | Copy of `Bay Playbook Landing v2.dc.html`. The landing page + master checklist. |
| `Bay Playbook Landing v2.dc.html` | — | Same file as `index.dc.html`. Use one, delete the other. |
| `Bay Playbook Decisions.dc.html` | `#decisions` | 10-question decision index. |
| `Bay Playbook Directory.dc.html` | `#directory` | Filterable database. Has an internal `#events` and `#vcs` anchor. |
| `Bay Playbook Guides.dc.html` | `#guides` | **Already a hash router itself** with 8 inner slugs. See §6. |
| `Bay Playbook Getting Around.dc.html` | `#around` | |
| `Bay Playbook Remote Setup.dc.html` | `#remote` | |
| `Bay Playbook Packing.dc.html` | `#packing` | |
| `Bay Playbook Apps.dc.html` | `#apps` | |
| `Bay Playbook Emergency.dc.html` | `#emergency` | |
| `Bay Playbook Housing.dc.html` | `#housing` | |
| `Bay Playbook SSN ITIN License.dc.html` | `#ssn` | |
| `Bay Playbook Culture.dc.html` | `#culture` | |
| `Bay Playbook Networking.dc.html` | `#networking` | |
| `Bay Playbook Groceries.dc.html` | `#groceries` | |
| `Bay Playbook Weekends.dc.html` | `#weekends` | |
| `Bay Playbook Accelerators.dc.html` | `#accel` | |
| `support.js` | — | **Required runtime.** Do not modify or replace. |

---

## 2. How these files work (read this before touching anything)

Each `.dc.html` is a self-contained page with three parts:

```html
<!DOCTYPE html><html><head>
  <script src="./support.js"></script>      <!-- the runtime -->
</head><body>
<x-dc>
  <helmet>…fonts + body resets…</helmet>    <!-- goes to <head> -->
  …template markup with {{ holes }}…
</x-dc>
<script type="text/x-dc" data-dc-script data-props="…">
  class Component extends DCLogic { … }     <!-- the logic -->
</script>
</body></html>
```

The template language is small:

- `{{ path }}` — dotted lookup only. **No expressions.** `{{ a + b }}` fails silently. Anything computed lives in `renderVals()` and is exposed by name.
- `<sc-for list="{{ items }}" as="item" hint-placeholder-count="6">` — loop. `$index` is in scope.
- `<sc-if value="{{ flag }}" hint-placeholder-val="{{ false }}">` — conditional.
- `renderVals()` returns a flat object of everything the template reads: values, arrays, and event handlers.
- Attributes: `x="{{ path }}"` passes the raw value; `x="a {{p}} b"` interpolates a string. Handlers are camelCase (`onClick`, `onChange`, `onError`).

**Styling is 100% inline `style=""` attributes.** There are no CSS classes anywhere by design. The only `<style>` content is in `<helmet>`: font imports, body reset, and `@media print`. Keep it that way — do not extract classes, do not add a stylesheet.

---

## 3. The design system (do not change any value)

### Fonts

Loaded from Google Fonts in every page's `<helmet>`:

```html
<link href="https://fonts.googleapis.com/css2?family=Archivo:wght@400;500;600;700;800;900&family=IBM+Plex+Mono:wght@400;500;600;700&family=Caveat:wght@600;700&family=Silkscreen:wght@400;700&display=swap" rel="stylesheet">
```

| Face | Role |
|---|---|
| **Archivo** | Everything structural. Body 13–17px/1.55–1.65. Headings 900 weight, `letter-spacing:-0.02em`. h1 38px, h2 20px, card titles 14.5px/800. |
| **IBM Plex Mono** | All eyebrows, labels, data, counts, prices, source chips. 9.5–12px, `letter-spacing:0.06–0.1em`, usually UPPERCASE. |
| **Caveat** | Pull quotes only (26–27px/700) and the "The" / "Playbook" in the wordmark. |
| **Silkscreen** | The word "bay" in the wordmark. Nothing else. |

> **If you bundle offline:** the Google Fonts link is a network fetch. Either inline the font files as base64 `@font-face` in the helmet, or accept the fallback. Do not swap in different families.

### Palette

```
--accent   #E8500A   coral — the single accent, on a CSS custom property
#FAF7F1    page background
#FFFDF8    card background (slightly warmer than the page)
#F7F3EA    muted / secondary card background
#F5F1E7    "completed" or "selected" row background
#F1EBDD    logo tile background, sidebar hover
#1A1712    ink — text, borders, dark panels
#3D3629    body text
#5A5243    secondary text
#8A8172    muted / label text
#B3A995    faintest label text
#C9C0AD    inactive chip borders, light text on dark
#E2DACA    standard card border, dividers
#EFE9DC    hairline dividers inside cards, empty track fill
#332D24    dividers inside dark panels
#F5B896    highlight text on dark panels
#7FD1A6    the only "good/positive" green (toggle dots, positive verdicts)
```

`--accent` is set at runtime by every page's logic:

```js
componentDidMount() { document.documentElement.style.setProperty('--accent', this.props.accent ?? '#E8500A'); }
componentDidUpdate() { /* same line */ }
```

Every page exposes an `accent` prop with options `["#E8500A","#6C8CF5","#0E7C7B","#1A1712"]`. Keep this.

### Component vocabulary

Reuse these exactly. They are the page's whole visual language.

**Mono eyebrow** — labels every card and section:
```html
<div style="font-family:'IBM Plex Mono',monospace;font-size:10.5px;letter-spacing:0.1em;color:#8A8172;margin-bottom:6px">WHAT THIS IS</div>
```

**Standard card**: `border:1px solid #E2DACA; background:#FFFDF8; border-radius:10px; padding:15px 17px`
**Emphasis card**: `border:1.5px solid #1A1712`
**Warning card**: `border:1.5px solid var(--accent)`
**Ink panel** (the "answer" of every interactive): `background:#1A1712; color:#FAF7F1; border-radius:12px; padding:22px 24px`

**Filter chip**:
```html
<button style="font-family:'IBM Plex Mono',monospace;font-size:12px;font-weight:500;padding:7px 12px;border-radius:999px;cursor:pointer;border:1px solid #C9C0AD;background:#FFFDF8;color:#3D3629" style-hover="border-color:#1A1712">Label</button>
```
Active state: `border:1px solid #1A1712; background:#1A1712; color:#FAF7F1`.

**Checklist row**:
```html
<label style="display:grid;grid-template-columns:20px 1fr;gap:12px;align-items:baseline;padding:12px 16px;border-bottom:1px solid #EFE9DC;cursor:pointer">
  <input type="checkbox" style="width:15px;height:15px;accent-color:#E8500A;cursor:pointer;position:relative;top:2px">
  <span style="font-size:14.5px;line-height:1.5">…</span>
</label>
```
Done state: row `background:#F5F1E7`, text `color:#8A8172; text-decoration:line-through`.

**Source chips** — every page ends with these:
```html
<a href="…" target="_blank" style="font-family:'IBM Plex Mono',monospace;font-size:12px;padding:6px 12px;border:1px solid #C9C0AD;border-radius:999px;color:#3D3629;background:#FFFDF8">label ↗</a>
```

**Logo tile** (used on Apps, Accelerators, Groceries) — a favicon over a monogram fallback:
```js
logoEl(domain, pad) {
  if (!domain) return null;
  return React.createElement('img', {
    src: 'https://www.google.com/s2/favicons?domain=' + domain + '&sz=64',
    alt: '', onError: e => { e.currentTarget.style.display = 'none'; },
    style: { position:'absolute', top:0, left:0, width:'100%', height:'100%',
             objectFit:'contain', background:'#FFFFFF', padding:pad, boxSizing:'border-box' },
  });
}
```
The monogram `<span>` sits behind it in the template; if the favicon fails, `onError` hides the image and the letter shows through. **This must be built in `renderVals()`, not the template** — see §8.

---

## 4. The shared shell

Every page has the identical shell. In the merged file this renders **once**, outside the route conditionals.

```html
<div style="display:grid;grid-template-columns:280px 1fr;min-height:100vh;max-width:1240px;margin:0 auto">
  <aside style="position:sticky;top:0;height:100vh;box-sizing:border-box;overflow-y:auto;padding:28px 24px 24px;border-right:1.5px solid #E2DACA;display:flex;flex-direction:column;gap:20px">
    <!-- wordmark -->
    <a href="#home" style="display:block;line-height:1;width:fit-content;color:#1A1712">
      <div style="font-family:'Caveat',cursive;font-size:19px;font-weight:700;transform:rotate(-5deg);margin-left:2px">The</div>
      <div style="font-family:'Silkscreen',monospace;font-size:27px;font-weight:700;color:var(--accent);text-shadow:2.5px 2.5px 0 #1A1712;letter-spacing:-1px;margin:-4px 0 -2px">bay</div>
      <div style="font-family:'Caveat',cursive;font-size:19px;font-weight:700;transform:rotate(-4deg);text-align:right">Playbook</div>
    </a>
    <!-- nav: 6 groups, see §5 -->
    <!-- Directory CTA pinned bottom with margin-top:auto -->
  </aside>
  <main style="padding:48px 56px 72px;box-sizing:border-box;min-width:0;max-width:900px">
    <!-- route content -->
  </main>
</div>
```

Each page's `<main>` opens with a stage/type badge row, then `h1`, then a lead paragraph:

```html
<div style="display:flex;align-items:center;gap:10px;margin-bottom:16px;flex-wrap:wrap">
  <span style="font-family:'IBM Plex Mono',monospace;font-size:11px;font-weight:600;letter-spacing:0.1em;color:var(--accent)">S2 · LANDING, WEEK 1</span>
  <span style="font-family:'IBM Plex Mono',monospace;font-size:10.5px;font-weight:600;letter-spacing:0.06em;background:#1A1712;color:#FAF7F1;padding:3px 8px;border-radius:4px">GUIDE</span>
  <span style="font-family:'IBM Plex Mono',monospace;font-size:11px;color:#8A8172">VERIFIED JUL 2026</span>
</div>
```

---

## 5. Navigation taxonomy (canonical — already in every file)

```js
[
  { label: 'S0 · DECIDING',          pages: [['Reality check','__reality'], ['Decision index','__decisions']] },
  { label: 'S1 · BEFORE YOU FLY',    pages: [['Visa guide','visa'], ['Remote setup','__remote'], ['Short-term housing','short-term-housing'], ['Packing + medications','__packing']] },
  { label: 'S2 · LANDING, WEEK 1',   pages: [['Day 0–7 checklist','day07'], ['SIM + phone','sim'], ['Apps to download','__apps'], ['Bank, cash & Clipper','bank'], ['Getting around','__around'], ['Emergency contacts','__emergency']] },
  { label: 'S3 · FIRST MONTH',       pages: [['Housing & neighbourhoods','__housing'], ['SSN, ITIN & licence','__ssn'], ['Money, credit & ITIN','money'], ['Healthcare & insurance','healthcare']] },
  { label: 'S4 · LIVING HERE',       pages: [['Culture & etiquette','__culture'], ['Networking in SF','__networking'], ['Indian groceries & food','__groceries'], ['What to do on weekends','__weekends']] },
  { label: 'S5 · COMPANY',           pages: [['US entity','entity'], ['Accelerators & programs','__accel']] },
]
```

- `__token` → a standalone page → becomes `#route` in the merged file.
- bare slug (`visa`, `sim`, `bank`, `day07`, `money`, `healthcare`, `entity`, `short-term-housing`) → lives **inside** the Guides router → becomes `#guides/visa` or `#guides` + inner state.
- `__reality` → stays an external URL: `https://thefounderfolks.com/bay-playbook/reality-check`

Nav item markup:
```html
<a href="{{ pg.href }}" style="display:flex;align-items:center;justify-content:space-between;gap:8px;padding:7px 10px;border-radius:7px;font-size:13.5px;font-weight:600;color:{{ pg.color }};background:{{ pg.bg }}" style-hover="background:#F1EBDD">{{ pg.title }}</a>
```
Active page: `color:#FAF7F1; background:#1A1712`. Everything else: `color:#1A1712; background:transparent`.

---

## 6. The Guides sub-router

`Bay Playbook Guides.dc.html` holds 8 pages in a `PAGES` object keyed by slug: `visa`, `short-term-housing`, `day07`, `sim`, `bank`, `healthcare`, `money`, `entity`. It renders whichever the hash points at, and each page is built from block helpers (`h()`, `p()`, `list()`, `steps()`, `table()`, `nobody()`, `friend()`, `note()`, `links()`).

Seven slugs were **deleted** from it because they were superseded by richer standalone pages. It has a `RETIRED` map that redirects them. In the merged file, translate that to route redirects:

```js
const RETIRED = {
  'remote-setup': 'remote', 'packing': 'packing', 'apps': 'apps',
  'getting-around': 'around', 'emergency': 'emergency',
  'housing': 'housing', 'culture': 'culture',
};
```

Any old `#remote-setup` link must land on `#remote`. Do not resurrect the deleted content.

---

## 7. Build the merged file

**Do it incrementally.** Verify after each batch so a failure never leaves you with a broken 7,000-line file.

### Pass 1 — router shell only

Create `Bay Playbook.dc.html`:

```js
class Component extends DCLogic {
  state = { route: 'home', /* per-page state added later */ };
  componentDidMount() {
    this.applyAccent();
    this.sync();
    window.addEventListener('hashchange', this.sync);
  }
  componentWillUnmount() { window.removeEventListener('hashchange', this.sync); }
  sync = () => {
    const raw = (location.hash || '#home').slice(1);
    const RETIRED = { 'remote-setup':'remote','packing':'packing','apps':'apps','getting-around':'around','emergency':'emergency','housing':'housing','culture':'culture' };
    const route = RETIRED[raw] || raw || 'home';
    this.setState({ route });
  };
  applyAccent() { document.documentElement.style.setProperty('--accent', this.props.accent ?? '#E8500A'); }
  componentDidUpdate() { this.applyAccent(); }
  renderVals() {
    const r = this.state.route;
    return {
      isHome: r === 'home', isAround: r === 'around', isRemote: r === 'remote',
      /* …one boolean per route… */
      navGroups: /* §5, hrefs as '#route' */,
    };
  }
}
```

Template: shell + sidebar + one placeholder `<main>`. **Confirm it loads and the nav switches state before going further.**

### Pass 2–5 — fold pages in, 3–4 at a time

- Batch A: `home`, `decisions`, `directory`
- Batch B: `around`, `remote`, `emergency`, `packing`
- Batch C: `housing`, `ssn`, `apps`, `groceries`
- Batch D: `culture`, `networking`, `weekends`, `accel`, `guides`

For each page:
1. Copy its `<main>` contents **verbatim** into the router template, wrapped in `<sc-if value="{{ isRoute }}" hint-placeholder-val="{{ false }}">`.
2. Copy its module-level constants (the `const APPS = […]`, `const CH = {…}` data arrays) to the top of the merged logic file. Rename on collision — several pages use `CHECKS`, `GLOSS`, `KEY`. Prefix them: `APPS_CHECKS`, `HOUSING_GLOSS`, etc.
3. Move its state into a namespaced slice: `state.apps = { … }` instead of top-level.
4. Move its `renderVals()` body into a method (`appsVals()`) and spread only the active route's values:
   ```js
   renderVals() {
     const r = this.state.route;
     const base = { navGroups: …, isHome: r === 'home', /* … */ };
     if (r === 'apps') return { ...base, ...this.appsVals() };
     if (r === 'housing') return { ...base, ...this.housingVals() };
     return base;
   }
   ```
   Only the active route's values need to resolve; the inactive `sc-if` branches render nothing.

### localStorage keys — keep these exactly

Users' saved progress lives here. Changing a key wipes it.

```
bp-checklist                 landing master checklist
bp-getting-around-checks     Getting around
bp-remote-setup-checks       Remote setup
bp-packing                   Packing
bp-apps                      Apps  (also stores gate toggles)
bp-emergency-card            Emergency (also blood group + contact fields)
bp-housing                   Housing
bp-ssn-dmv                   SSN, ITIN & licence (also visa/spouse selection)
bp-culture-checks            Culture
bp-networking                Networking (also the 70-word DM draft)
bp-groceries                 Groceries
bp-weekend-checks            Weekends
bp-accelerators              Accelerators
```

### Pass 6 — bundle

The page already has a `<template id="__bundler_thumbnail" data-bg-color="#FAF7F1">` in `index.dc.html` — carry it over. It's the splash shown while the bundled file unpacks.

Inline everything: `support.js`, the Google Fonts CSS **and** the font files, and the one hero image (`framerusercontent.com/…53D5YShf24JrgYPgiJngc0hKkA.png` on the landing page). Favicon URLs in the logo tiles are `google.com/s2/favicons` requests — they will not resolve offline, which is fine: the monogram fallback handles it. If you want them offline, fetch each and inline as data URIs.

---

## 8. Four bugs we already hit — don't repeat them

**1. Inline `onError` in streamed markup gets evaluated as JS.**
Never write `onError="{{ handler }}"` on an `<img>` in the template. Before the runtime hydrates, the browser parses that as a literal `onerror` attribute and evaluates the string `{{ handler }}` — which is syntactically valid JS and throws `ReferenceError`. Build the whole `<img>` in `renderVals()` via `React.createElement` and render it as `{{ p.logo }}`. This is why `logoEl()` exists.

**2. `repeat(auto-fit, minmax(N,1fr))` silently collapses to one column.**
Two tracks need `2×N + gap ≤ available width`. The content column is ~517px wide (main 629px − 56px padding × 2), and inside a dark panel with 24px padding it's ~469px. Card grids use `minmax(240px,1fr)` in main and `minmax(220px,1fr)` inside panels. Raising those floors makes the page twice as tall.

**3. `white-space:nowrap` on a flex item overflows the card.**
Flex items default to `min-width:auto`, so a `nowrap` label can neither shrink nor wrap, and `overflow:visible` lets it escape into the next grid column. Don't put `nowrap` on variable-length strings. If a label won't fit beside a title at ~215px of header width, move it above the title as a mono eyebrow.

**4. Tooltips need a measured side-flip.**
The glossary tooltips are `position:absolute` panels 280px wide. Anchored `left:0` on a trigger near the right edge, they overflow the viewport and make the document horizontally scrollable. `max-width:calc(100vw - 44px)` does **not** help — it clamps against viewport width, not the trigger's distance from the edge. Measure at open time:
```js
toggle: e => {
  let align = 'left';
  const r = e.currentTarget.getBoundingClientRect();
  if (r.left + 288 > window.innerWidth && r.right - 288 > 0) align = 'right';
  this.setState({ term: k, align });
}
```
then drive `left`/`right` from state (`'0'` / `'auto'`), and set `box-sizing:border-box` on the panel so the width is the real outer width.

---

## 9. What each page's interactive element does

Preserve all of these. They are the point of each page.

| Route | Interaction |
|---|---|
| `home` | Master checklist, 5 stages, filter by "Everything / A friend will tell you / Nobody tells you". Persists. |
| `around` | Trip picker (10 chips → take/cost/time panel); Muni pass slider (round trips → pay-per-ride vs $86 pass, two bars + verdict); checklist. |
| `remote` | **Flight-date sequencer** — set your departure date and 9 lead-time tasks resolve to real start/latest dates with OVERDUE / START NOW states; TCS calculator (₹ slider → one transfer vs split across 1 April); checklist. |
| `packing` | **Meds sorter** — tick what you'd pack, get suitcase-vs-buy-there columns, paperwork tags, and a count of letters + 90-day supplies needed. Microclimate strip: Ocean Beach 52°F vs Mission 65°F on one axis. |
| `apps` | **Dependency gate** — 3 switches (US number / bank account / driving) × 9 category filters resolve 45 apps into installable-now (grouped by when) vs a dark "NOT YET" panel naming each blocker. iOS + Android store links, favicon logos. |
| `emergency` | **Lock-screen card** — pick your city, card fills with that city's dispatch + non-emergency line alongside 911/988/poison control/consulate, all `tel:` links, plus editable blood group and contact. ER-or-urgent-care triage picker. |
| `housing` | **Region matcher** — where your week happens + who's moving + rent-ceiling slider → names one of 5 regions with 3 neighbourhoods, tradeoff, runner-up. Rent range chart with your ceiling drawn across it. Year-two rent calculator (rent slider × 3 building-age tiers). Credit-door matcher. |
| `ssn` | **Branch selector** — visa + spouse visa → SSN verdict for both and what changes. Landing-date sequencer with the 10-day licence deadline. 16 acronym tooltips + a full glossary panel. |
| `culture` | Two-founder before/after toggle; **culture map** plotting India vs US on explicit-vs-implicit × blunt-vs-cushioned; **the decoder** (investor / at-work switch → meaning, temperature, what to do); month-three diaspora counter. |
| `networking` | **Channel matcher** — what you're short of + founder/operator + SF/South Bay → 2–3 channels from the 8-channel map. **IST collision clock** — SF events 6–9pm against your India sync, with a toggle that moves the sync to 7am. Live 70-word cold-DM counter, persists. |
| `groceries` | **Trip planner** — tick 13 items + SF/South Bay + car + Costco membership → routes each item and tells you how many trips you actually need. |
| `weekends` | **Saturday shuffler** — car + who's coming filters 32 things, then builds a morning/afternoon/evening itinerary you can re-roll. Booking clock (live dates from today). Drive-time chart with traffic overlay. |
| `accel` | **Program filter** — stage + sector + constraint over 45 programs, sector-specific floated above generalists. Deadline countdowns. "What one point of your company buys" chart (cash ÷ equity, 18 programs). Favicon logo wall. |
| `decisions` | 10-question decision index. |
| `directory` | Filterable database, `#events` and `#vcs` anchors. |
| `guides` | Inner router over 8 remaining slugs. |

---

## 10. Verification before you call it done

- Console clean on load — no `ReferenceError`, no failed resource loads, no unresolved `{{ }}` in the DOM.
- Every sidebar entry in all 6 groups navigates and highlights the active page.
- Old hashes redirect: `#packing`, `#apps`, `#housing`, `#culture`, `#emergency`, `#remote-setup`, `#getting-around`.
- Browser back/forward moves between routes.
- No horizontal scroll at any route, tooltips open included: `document.documentElement.scrollWidth === clientWidth`.
- Card grids render 2-up, not 1-up.
- Checklists persist across a reload, and the 13 localStorage keys are unchanged.
- `--accent` resolves to `#E8500A` everywhere; it should be the only custom property referenced.
- Works from `file://` with no network — fonts may fall back, favicons will show monograms, everything else must function.

---

## Alternative: port to a real codebase

If you'd rather have a maintainable app than one bundled file, the sane port is Vite + React with one route per page and the same inline-style approach (or CSS modules if you must). The DC template maps mechanically: `sc-for` → `.map()`, `sc-if` → `&&`, `{{ hole }}` → `{value}`, `renderVals()` → the component body. Keep §3's tokens, §5's taxonomy, and §8's four gotchas — those are the parts that took the longest to get right.
