# CLAUDE.md

Context for future Claude sessions working on this repo.

## What this is

Personal portfolio of Omar Faruk — product engineer with a security habit. Single static HTML file, no build step, no framework. Deployed on Vercel at https://omarfaruk.bd.

- **Entry:** `index.html` (everything inlined — CSS, JS, base64 portrait)
- **Assets:** `assets/testimonial-*.{jpg,jpeg}` (testimonial avatars), `og.png` (social card)
- **Config:** `vercel.json` (clean URLs, security headers, OG cache)
- **Analytics:** onedollarstats via `stonks.js` (loaded from CDN in `index.html`)

## Design system

Terminal / editorial aesthetic. Dark by default, light mode via `[ light mode ]` button in the topbar.

- **Fonts:** IBM Plex Mono (body/UI), Instrument Serif (display), Source Serif 4 (testimonials/prose)
- **Accent:** phosphor-green `#8fd694` (dark) / `#3a9853` (light)
- **Layout:** CSS Grid with `grid-template-areas`, responsive breakpoints at 780px and 560px
- **Theme toggle:** View Transition API with `clip-path` wipe, stored in `localStorage['omar.theme']`
- **Cursor:** custom two-tone green block (inline SVG data URI on `html, body`)

## Page structure

```
intro-gate (neofetch terminal, unlocks audio)
shell
  topbar                              brand + meta + theme toggle
  hero-area                           flex column, min-height ~viewport
    cmd                               "~/omar $ whoami --verbose" (types in)
    whoami                            grid: name/bio/idcard (centered via margin: auto 0)
    now-strip                         ticker "$ tail -f now.log · uptime · reading · shipping · ..."
  scroll-hint                         floating "scroll ↓" button, fixed, fades on scroll
  section#testimonials                3 real quotes w/ avatars
  section#work                        work list (klasio, tinkerflow, sms-trap, sawn, take-a-break)
  section#focus (3 sub-strips)
    03.1 engineering                  vim-card (line gutter, modeline)
    03.2 security                     auth-box (ok/warn bullets, tags)
    03.3 fundamentals                 stack-card (user/syscall/kernel/driver/hw layers)
  section#writing                     writing wip notice
  section#contact                     email, github, linkedin, etc.
  footer                              hash + konami hint
```

## Intro flow (first load)

Staged sequence gated on user gesture (browser autoplay policy blocks audio until a real click/key):

1. **Gate** — neofetch terminal overlay (`$ neofetch` → ASCII OMAR → tagline → prompt). Dismissed on any `pointerdown` / `keydown` / `touchstart` / `click` / `mousedown` via the `startIntro()` function.
2. **Typing** — `.cmd-typed` reveals `whoami --verbose` char-by-char (0.85s, 16 steps). Keystroke ticks scheduled in JS to match.
3. **Reveal** — `.reveal-in` elements fade up with staggered `animation-delay: calc(var(--i) * 110ms)`.
4. **Portrait scan** — idcard portrait runs a scramble→scan→reveal animation, triggered ~2.2s after intro start.
5. **Border fade-in** — section rules fade in 1s after intro completes (prevents stray borders during reveal).

Reduced-motion users skip the gate entirely and get instant content.

## Audio

All Web Audio — no files.

- `playClick()` — theme toggle (noise burst through bandpass + sine body)
- `playKey()` — keystroke ticks (short noise bursts)
- `playHover()` — subtle UI blip (sine sweep 1700→1200 Hz, 80ms, gain 0.045)

Hover sounds are attached via a single delegated `mouseover` listener on `document`, scoped to:
`.work li[onclick]`, `.ch-grid a`, `.quote`, `.switch`, `[data-scroll-next]`, `.writing li`, `.auth-box .row`, `.vim-card .vim-line:not(.empty)`, `.stack-card .stack-layer`, `.stack-card .stack-now`. 55ms throttle prevents machine-gun sounds on fast sweeps. Skipped for `prefers-reduced-motion`.

## Key patterns & gotchas

**Light-mode vars apply to both `<html>` and `<body>`.** The early pre-paint script reads `localStorage['omar.theme']` and adds `light` to `<html>` before first paint — otherwise the html background stays dark while the body turns light, which leaks through during `intro-loading` (when `overflow: hidden` collapses body height). `applyTheme()` toggles `light` on both elements.

**ASCII art in the gate is injected from a JS string** (`#gateAscii`). HTML formatters re-indent `<pre>` contents, which destroys alignment. Injecting from a JS array-join-string is formatter-proof.

**Hero centering:** `.hero-area { display: flex; flex-direction: column; min-height: calc(100svh - 135px) }`. The `.cmd` pins top, `.whoami` centers via `margin: auto 0`, `.now-strip` pins bottom. `svh` (not `vh`) avoids mobile browser-chrome jitter.

**Scroll hint is `position: fixed` and hides on scroll** via `is-hidden` class toggled when `scrollY > max(120, 25vh)`. Semi-transparent bg + `backdrop-filter: blur(6px)` so it reads against any section.

**Halftone portrait** renders on a canvas via `renderHalftone(CELL)` — source image is inlined as base64. Cell size is 5. Re-renders on theme change.

**Konami code easter egg** — up/up/down/down/left/right/left/right/b/a reveals a `<template>`-stored flag. Signature click (`#sig`) also triggers it.

**Intro-loading state** uses `display: none` (not `visibility: hidden`) on hidden elements to prevent stray border/layout gaps before content reveals.

## Deploy

Static on Vercel. The `redesign/terminal-system` branch (PR #1) was the redesign baseline. Merges to `main` auto-deploy.

```sh
# Local preview
python3 -m http.server 8080
# or
bunx serve .
```

## Screenshot / OG image

`og.png` is generated by a Playwright script that captures the hero (cmd + whoami) with topbar and ticker hidden. If regenerating:

```sh
# In a separate dir:
bun add playwright
node capture.mjs   # see prior chat for the script; viewport 1200x630, deviceScaleFactor 2
```

Recent version is 2400×1260 retina — most OG consumers accept this fine; downscale to 1200×630 if strict compatibility needed.

## Content sources

- **Testimonials:** real quotes from M. Mahbubur Rahman (iViveLabs CTO), Anis Uddin Ahmad (ajaxray.com), Ahmed Shamim Hassan. Avatars are in `assets/`.
- **Work list:** Klasio, Tinkerflow, sms-trap, sawn, Take-a-Break.
- **Currently reading:** OSTEP (Operating Systems: Three Easy Pieces).
- **Principles (03.1):** 6-rule vim buffer. Keep copy plain/grounded — user dislikes marketing-speak (e.g., "code that doesn't break on a Friday evening" > "boring reliable releases").

## Future-Claude notes

- Don't add documentation files unless asked.
- Keep copy concrete and plain — no marketing-speak, no buzzwords.
- When adding new hoverable elements, add their selector to the `hoverSel` list in the hover-sound IIFE.
- When adding sections, include them in `.section, .contact, .testimonials, ...` border-fade-in selector list.
- When wrapping ASCII or any whitespace-sensitive content in `<pre>`, prefer JS injection over inline HTML to survive formatters.
- Portrait uses fixed `CELL = 5`. Don't oversample on mobile — it makes the halftone noisy (user complained about this repeatedly).
