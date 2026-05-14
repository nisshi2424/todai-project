# CLAUDE.md — todai-project

## Project Overview

A single-page Japanese HTML presentation and feedback form built for **Shoma** (奨真) to pitch an AI-powered study-efficiency system to his family as part of preparing for the **University of Tokyo (東京大学) 文科2類** entrance exam via the 共通テスト (National Common Test).

The document was co-created with Claude (Anthropic) and presents:
1. A 12-slide pitch deck explaining how AI can analyze 20 years of 共通テスト past papers
2. An embedded feedback form (for "俊希" the student and "父" the father) that POSTs answers to a Google Apps Script endpoint

## Repository Structure

```
todai-project/
├── index.html      # Entire application — slides, CSS, JS, and form
└── robots.txt      # Disallow: / (blocks all crawlers)
```

There is no build system, no package manager, no dependencies, and no server-side code. Everything is self-contained in `index.html`.

## Tech Stack

| Layer | Choice |
|---|---|
| HTML | Plain HTML5, `lang="ja"` |
| CSS | Inline `<style>` block, custom properties (CSS variables) |
| JavaScript | Vanilla JS in `<script>` tags, no frameworks |
| Fonts | Google Fonts — Noto Sans JP + Inter |
| Form backend | Google Apps Script (external) |
| Hosting | Static file served as-is |

## Key Conventions

### CSS Architecture
- All styles live in a single `<style>` block in `<head>`
- Design tokens are defined as CSS custom properties on `:root` (e.g., `--dark`, `--blue`, `--gradient-hero`)
- Dark theme only; color palette: near-black backgrounds, white text, blue/violet/cyan accents
- Slide layout: `.slide` class with `page-break-after: always` for print/PDF support
- Responsive breakpoint at `max-width: 768px` with a full mobile override section
- Print styles at `@media print`

### JavaScript Conventions
- No modules, no imports — all functions are global
- State is held in a single module-level variable: `let currentRespondent = null`
- Form submission uses `fetch()` with `mode: 'no-cors'` to the Google Apps Script URL
- Fallback: if fetch fails, the serialized answers are copied to the clipboard via `navigator.clipboard`
- Progress bar updates on every `input` event via event delegation on `document`

### Content Language
- All user-visible text is in **Japanese**
- Comments in HTML are also in Japanese where they exist
- Variable names and function names are in English

## Password Gate

The page has a client-side password gate. The password (`todai2026`) is hardcoded in the JavaScript:

```js
if (document.getElementById('pwInput').value === 'todai2026') { ... }
```

This provides only visual/UX gating — it offers no real security since the source is public. The `robots.txt` (`Disallow: /`) discourages indexing.

## Form Submission

The form POSTs JSON to a Google Apps Script web app URL stored in `SCRIPT_URL`:

```js
const SCRIPT_URL = 'https://script.google.com/macros/s/AKfycb.../exec';
```

Payload shape:
```json
{
  "timestamp": "<ISO 8601>",
  "answers": [
    {
      "respondent": "俊希 | 父",
      "question": "Q1 — <full question text>",
      "answer": "<user input>"
    }
  ]
}
```

The form shows only answered questions (skips blanks) to keep the payload lean.

## Development Workflow

There is no build step. To make changes:

1. Edit `index.html` directly
2. Open it in a browser to verify (no server needed — `file://` works)
3. Commit and push

For layout/responsive testing, resize the browser window below 768 px or use browser DevTools device emulation.

For print/PDF testing, use browser print preview (`Ctrl+P` / `Cmd+P`).

## Git Conventions

Commit messages in this repo have been short imperative phrases:
- `Update index.html`
- `Create robots.txt`

The file was originally named `申し送り及び質問状.html` before being renamed to `index.html`.

## Active Branch

Development happens on `claude/add-claude-documentation-PFFub`. Push all changes there.

## Things to Be Aware Of

- **No tests** — there is no test suite. Verify changes manually in a browser.
- **No linter/formatter** — no ESLint, Prettier, or HTML validator configured. Keep style consistent with the surrounding code.
- **Password is client-side only** — do not treat it as a real access control mechanism.
- **Google Apps Script URL is public in source** — treat it as semi-public; do not put secrets in the payload.
- **All content is in Japanese** — maintain Japanese for any user-visible strings added to slides or form labels.
- **Single-file architecture** — resist the urge to split CSS/JS into separate files unless the project outgrows the current scope; the self-contained single-file approach is intentional for easy sharing.
