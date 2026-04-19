# Manage AI Remote™ — Pixel-Perfect Clone Build

## Mission
Clone the Manage AI Remote™ app into a pixel-perfect static replica, committed to GitHub, deployable to Vercel. The clone must match the source **visually** while conforming to the Remote™ Design Specification.

---

## Sources

**Primary source — the app to clone (visual truth for layout/screens):**
- URL: https://manageai.netlify.app
- What it is: The live Remote™ dashboard (Manager Surface: Briefing, Desk, Team, 1:1; plus Operations and Control sections).
- Role: Source of truth for **structure, layout, screens, navigation, copy**.

**Design specification — how the clone should conform (system truth):**
- URL: https://manageai-remote.vercel.app
- What it is: Remote™ Design Specification v2.1 (Mar 25, 2026). Six docs covering Vision, Workflows, IA, Design System, Controls & SDK, State & Data.
- Role: Source of truth for **colors, typography, spacing, component rules, do/don't rules, navigation behavior, view-toggle logic**.

**Conflict rule:** When the Netlify app contradicts the Vercel spec, the spec wins. Log every conflict in `CONFLICTS.md` with screenshots and the chosen resolution.

---

## Pixel-Perfect Contract

- **Breakpoints:** 1920×1080, 1024×768, 390×844
- **Target:** <1% pixel diff per breakpoint (via `pixelmatch`)
- **Hard cap:** 6 diff iterations per breakpoint, then stop and surface remaining gaps
- **Animations:** Frozen via `page.emulateMedia({ reducedMotion: 'reduce' })` during capture
- **Fonts:** Await `document.fonts.ready` before every screenshot
- **Lazy loads:** Slow scroll-to-bottom pass before capture, then scroll back to top

---

## Design System

See **`design-tokens.css`** in repo root — tokens are extracted from Doc 4 of the spec. Reference tokens via CSS custom properties, don't hardcode hex values.

### Do (spec rules)
- White dominates 60–70%+ of every layout
- Blue is accent only — headings, icons, borders, badges
- Sans-serif only — Montserrat for headings, Inter for body
- ALL CAPS only for tiny section labels (10px, 2.5px tracking, Slide Blue)
- Left-align body text, center only hero sections
- Cards 12px radius, buttons 6px radius
- Alternate White / Snow sections for rhythm
- Icons: Lucide line style only
- Button hierarchy: Primary → Ghost → Link

### Don't (spec rules)
- No pure black (`#000`) for body — use Navy `#0F172A`
- No serif fonts ever
- No ALL CAPS titles or body
- No gradients (subtle Navy CTA glow excepted)
- No 3D, no ornate icons
- No heavy drop shadows on cards
- No obviously AI-generated imagery
- Don't style destructive actions red by default

---

## Build Constraints

- Static files only. HTML + CSS + JS. No framework rewrite.
- Preserve class names, DOM order, and inline styles from the Netlify source.
- Self-host every asset **except** Google Fonts (Montserrat + Inter).
- Leave Cloudflare email obfuscation (`/cdn-cgi/l/email-protection`) as-is.
- Commit after every numbered build step with a clear message.

---

## Output Structure

```
/
  index.html
  pages/                  (multi-page routes if present)
  assets/
    css/
    js/
    fonts/
    img/
  design-tokens.css       (pre-supplied — do not overwrite)
  CLAUDE.md               (this file)
  CONFLICTS.md            (spec vs. source conflicts, as they arise)
  README.md               (final: sources, diff scores, regen instructions)
  scripts/
    capture-source.js
    capture-clone.js
    diff.js
  reference/              (source screenshots — COMMITTED)
  clone/                  (clone screenshots — gitignored)
  diff/                   (diff images — gitignored)
```

---

## Build Steps (execute in order)

1. **DETECT** rendering mode of `manageai.netlify.app` (SSG vs SPA). Report framework signals (`__NEXT_DATA__`, `__NUXT__`, `/_next/`, `/assets/index-*.js`, response headers).
2. **MIRROR** the source. `wget` if SSG. Playwright route-crawler if SPA. Download every referenced asset.
3. **NORMALIZE** — restructure into the output tree above. Rewrite absolute URLs to relative. Self-host external assets (except Google Fonts).
4. **INJECT TOKENS** — link `design-tokens.css` before all other stylesheets in every HTML file. Don't edit the tokens file.
5. **CAPTURE REFERENCE** — run `scripts/capture-source.js` against the live Netlify URL at all three breakpoints. Commit `/reference/`.
6. **CAPTURE CLONE** — serve locally (`npx http-server . -p 8080 -s`), run `scripts/capture-clone.js`. Output to `/clone/`.
7. **DIFF LOOP** — `scripts/diff.js` produces `/diff/{bp}.png` + percentage. Per breakpoint: analyze failing regions, fix, re-screenshot, re-diff. Hard cap: 6 iterations. Print PASS/FAIL table after each round.
8. **GIT + GITHUB** — `gh repo create manageai-remote-clone --private --source=. --remote=origin --push`. README.md documents sources, final scores, regen steps.
9. **DEPLOY PREP** — add `vercel.json` only if needed. Print literal command: `vercel --prod`. **Do not deploy.**

---

## Stop Conditions

Halt and surface to Brian if:
- Netlify source is auth-gated or non-200 on primary routes
- Any breakpoint fails to reach <1% diff after 6 iterations
- An asset can't be self-hosted (CORS, auth, or 404)
- A spec-vs-source conflict requires a judgment call you can't make safely
- Any step requires a credential or secret not already in the environment

---

## Known Context (Brian's environment)

- Mac mini, ARM, username `brianreinhart`
- `gh` CLI is authed
- Claude Code runs with `--dangerously-skip-permissions`
- Vercel account is connected
- Phoenix Creative Works / ManageAI branding is separate — this build does **not** rebrand. Preserve Manage AI Remote™ branding as-is from the source.
