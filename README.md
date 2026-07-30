# The Bay Playbook

A hands-on operating manual for Indian founders relocating to San Francisco. Twenty guides, sequenced across five stages — deciding, before you fly, landing week one, first month, and living here.

**Live site**: _(Netlify URL will go here after first deploy)_

## What this repo is

Seventeen self-contained HTML pages plus a shared runtime (`support.js`) and a single handoff document (`HANDOFF.md`). Each page is a full, standalone product with its own interactive: a flight-date sequencer, a Muni-pass calculator, an apps dependency gate, a visa-branch selector, a culture map, and so on.

No build step. No framework config. Every page opens directly in a browser.

## Quick start

```bash
git clone https://github.com/<owner>/bay-playbook.git
cd bay-playbook
python3 -m http.server 3000
open http://localhost:3000
```

Or open `index.html` directly. The pages are self-contained; `file://` works too (favicons will fall back to monograms, everything else is fine).

## Repo layout

```
.
├── index.html                       # Landing page (copy of Landing v2)
├── index.dc.html                    # Source of the landing (identical)
├── Bay Playbook <Route>.dc.html     # 15 other route pages + Directory + Guides
├── support.js                       # DC runtime (React + template DSL)
├── HANDOFF.md                       # Original build brief (design system, gotchas)
├── CLAUDE.md                        # Codebase context for Claude Code
├── netlify.toml                     # Netlify static-site config
└── LICENSE                          # MIT
```

## How the pages work

Each `.dc.html` file has three parts:

```html
<script src="./support.js"></script>   <!-- the runtime -->
<x-dc>
  <helmet>…fonts + body reset…</helmet>
  …template markup with {{ holes }}…
</x-dc>
<script type="text/x-dc" data-dc-script data-props="…">
  class Component extends DCLogic { … }
</script>
```

The runtime loads React from unpkg and hydrates the `<x-dc>` template. **Styling is 100% inline `style=""`** — see `HANDOFF.md` for the full design system.

## Contributing

1. Fork this repo.
2. Edit any `.dc.html` file directly — no build step needed.
3. Refresh the page to see changes.
4. Open a PR describing what changed and which page.

The four bugs already documented in `HANDOFF.md` §8 are the ones that took the longest to solve. Read that section before touching the runtime.

## Deployment

Netlify auto-deploys from `main` when connected via the Netlify UI, or run:

```bash
npx netlify-cli deploy --prod --dir=.
```

## License

MIT. See `LICENSE`.
