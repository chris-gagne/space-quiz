# Copilot instructions

## Project constraints

- The entire app is one self-contained `index.html` (inline `<style>` and `<script>`). Keep it that way: no external files, CDNs, frameworks, build step, or server. It must work when opened via `file://`.
- Accessibility, keyboard navigation, `prefers-color-scheme`, and `prefers-reduced-motion` support are requirements, not extras.

## Validation

There is no build, test, or lint tooling. To syntax-check the inline script:

```powershell
$h = Get-Content index.html -Raw; [regex]::Match($h, '(?s)<script>(.*)</script>').Groups[1].Value | Set-Content $env:TEMP\q.js; node --check $env:TEMP\q.js
```

Behavior must be verified by opening `index.html` in a browser.

## Architecture

- Three `<section>` screens (`#start`, `#quiz`, `#results`) live in one `<main>`; `show(name)` toggles their `hidden` attribute. Flow: `start()` → `renderQuestion()` → `choose(i)` → `next()` → … → `showResults()`.
- Quiz content is the `QUESTIONS` array: `{ q: question, a: [4 answers], c: correctIndex, f: fact shown after answering }`. JS-rendered counters, progress, and results derive from `QUESTIONS.length`, but the static markup (`aria-valuemax="10"`, "Question 1 of 10", start-screen copy) is hard-coded and must be updated if the count changes.
- State is three closure variables in the IIFE: `index`, `score`, `answered`. `answered` gates both clicks and the global `keydown` handler (`1`–`4` to answer, `Enter` to advance).
- The results tier (emoji, its `aria-label`, message) is chosen by score percentage in `showResults()`; the score display is animated by `countUp()`.

## Conventions

- **Theming:** all colors are CSS custom properties defined in `:root` and overridden in `@media (prefers-color-scheme: dark)`. Add new colors to both blocks; don't hard-code colors in rules (the progress-bar gradient is the deliberate exception).
- **Motion:** a global `prefers-reduced-motion` rule neutralizes CSS animations/transitions. JS-driven animation (e.g. `countUp`) must check `matchMedia("(prefers-reduced-motion: reduce)")` itself and jump to the final state.
- **Restarting CSS animations:** remove the class/animation, force reflow with `void el.offsetWidth`, then re-apply (used for the score bump and emoji pop).
- **Screen-reader announcements** go through the single `#live` polite region as full sentences; don't announce intermediate animation frames.
- **Focus management:** focus moves to the question heading on each render, to the Next button after answering, and to the results heading at the end (these headings have `tabindex="-1"`).
- **Answer state** is conveyed by more than color: `.correct`/`.wrong` classes plus ✓/✗ marks and an updated `aria-label`.
- Insert user-visible strings with `textContent`, not `innerHTML` interpolation.
