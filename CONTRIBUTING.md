# Contributing

Thanks for helping improve the Bay Playbook.

## Ground rules

- **No build step.** Edit `.dc.html` files directly. If a change would need a bundler, open an issue first.
- **No CSS classes.** The design system is 100% inline `style=""` attributes. See `HANDOFF.md` §3 for the tokens.
- **Do not touch `support.js`** unless you're rebuilding the runtime.
- **Do not rename any `bp-*` localStorage key.** Users' saved progress lives there.

## Workflow

1. Fork the repo (top-right of GitHub).
2. Clone your fork, create a branch: `git checkout -b fix-<page>-<what>`.
3. Edit the relevant `.dc.html` file. Refresh your browser to test.
4. Verify with the checklist below.
5. Commit and push: `git push origin fix-<page>-<what>`.
6. Open a pull request against `main` on the upstream repo. In the description, name the page and the specific fix.

## Pre-PR checklist

- [ ] Page opens with no console errors on `python3 -m http.server 3000`.
- [ ] No unresolved `{{ }}` visible in the rendered DOM.
- [ ] Sidebar links still navigate correctly.
- [ ] `document.documentElement.scrollWidth === document.documentElement.clientWidth` (no horizontal scroll).
- [ ] If you touched any `localStorage` reads/writes, keys are unchanged.
- [ ] Fonts, colors, and card recipes match `HANDOFF.md` §3.

## Reporting bugs

Open an issue with:
- The page (e.g. `Bay Playbook Apps.dc.html`).
- What you expected vs. what happened.
- Browser + OS.
- Screenshot if visual, console log if runtime.

## Adding a new guide

See `CLAUDE.md` → "Adding a page" for the full checklist. In short: duplicate a similar page, replace content, add a nav entry to `index.html`'s `pageIndex`, and update `HANDOFF.md` §5.

## Style of change

Small, page-scoped PRs are easier to review. If a change touches the shell, the sidebar, or the design tokens, call it out in the PR title (`[shell]`, `[tokens]`, `[sidebar]`).
