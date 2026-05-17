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

## HTML Document Structure (top to bottom)

```
<head>  Google Fonts link + single <style> block
<body>
  #passwordGate     fixed overlay shown on load; hidden after correct password
  .ai-topbar        sticky banner crediting Claude/Anthropic
  SLIDE 1: COVER    cover slide
  SLIDE 2: WHY AI   AI capability stats
  SLIDE 3: PROBLEM  problem statement
  SLIDE 4: B→A      before/after comparison
  SLIDE 5: VISION   knowledge-search-engine concept demo
  SLIDE 6: HOW      5-step workflow diagram
  SLIDE 7: SCOPE    subject priority table
  SLIDE 8: OUTPUT   output format options (grid)
  SLIDE 9: COST     cost & timeline cards
  SLIDE 10: INPUT   transition slide leading to the form
  SLIDE 11: CLOSING next-steps guide
  SLIDE 12: RESPOND inline feedback form (俊希 / 父)
  #successOverlay   fixed overlay shown on successful submit
  <script>          all JavaScript — two separate <script> blocks
                    (checkPw is in the first; everything else in the second)
```

## Slide Map

| # | Label in HTML | Japanese heading | Key component classes |
|---|---|---|---|
| 1 | COVER | AIで過去問を丸裸にする。 | `.cover`, `.ai-fact-strip`, `.cover-meta` |
| 2 | WHY AI | この資料を作ったAIの実力を知ってほしい。 | `.iq-compare-bar`, `.why-ai-grid` |
| 3 | PROBLEM | 共通テスト対策の9割は、力技だ。 | `.big-msg`, `.stat-row` |
| 4 | BEFORE→AFTER | 同じ時間で、圧倒的な差がつく。 | `.compare`, `.compare-old`, `.compare-new` |
| 5 | VISION | つくるのは、知識の検索エンジン。 | `.demo-box`, `.demo-entry` |
| 6 | HOW IT WORKS | AIが、5つのステップで解析する。 | `.flow-vertical`, `.flow-item` |
| 7 | SCOPE | まずは共通テストから。 | `.memo-box`, `.scope-table` |
| 8 | OUTPUT | 出力は、勉強スタイルに合わせて選べる。 | `.output-grid`, `.output-card` |
| 9 | COST | すでに手の届くところにある。 | `.cost-row`, `.cost-card` |
| 10 | NEXT STEP | ここからは、二人の情報が必要だ。 | `.big-msg`, `.stat-row` (reused) |
| 11 | NOTE | 回答が揃い次第、すぐに動き出す。 | `.closing-guide` |
| 12 | RESPOND | 回答はここから。 | `.form-section` (full form) |

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
- Design tokens are defined as CSS custom properties on `:root` (see table below)
- Dark theme only; color palette: near-black backgrounds, white text, blue/violet/cyan accents
- Slide layout: `.slide` class with `page-break-after: always` for print/PDF support
- Two slide background variants: `.slide-dark` (`--dark`) and `.slide-dark-2` (`--dark-2`)
- Responsive breakpoint at `max-width: 768px` with a full mobile override section
- Print styles at `@media print`; page size forced to A4 via `@page { size: A4; margin: 0; }`

### CSS Design Tokens (`:root`)

| Variable | Value | Usage |
|---|---|---|
| `--dark` | `#09090b` | Default slide background |
| `--dark-2` | `#0c0c10` | Alternate slide background |
| `--dark-surface` | `#141418` | Raised surface background |
| `--dark-card` | `rgba(255,255,255,0.04)` | Card / panel fill |
| `--dark-border` | `rgba(255,255,255,0.08)` | Card / panel border |
| `--white` | `#fafafa` | Primary text |
| `--white-70` | `rgba(250,250,250,0.7)` | Secondary text |
| `--white-50` | `rgba(250,250,250,0.5)` | Muted text |
| `--white-30` | `rgba(250,250,250,0.3)` | Placeholder / label text |
| `--white-10` | `rgba(250,250,250,0.10)` | Hairline tint |
| `--blue` | `#3b82f6` | Primary accent |
| `--violet` | `#8b5cf6` | Secondary accent |
| `--cyan` | `#06b6d4` | Tertiary accent |
| `--amber` | `#f59e0b` | Warning / highlight accent |
| `--gradient-hero` | `135deg, cyan→blue→violet` | Headings, badges, key numbers |
| `--gradient-blue` | `135deg, blue→violet` | Buttons, flow step numbers, progress bar |

**Gradient text technique** — used extensively throughout the file for headings and accent text:
```css
background: var(--gradient-hero);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
background-clip: text;
```

### JavaScript Conventions
- No modules, no imports — all functions are global
- State is held in a single module-level variable: `let currentRespondent = null`
- Form submission uses `fetch()` with `mode: 'no-cors'` to the Google Apps Script URL
- Fallback: if fetch fails, the serialized answers are copied to the clipboard via `navigator.clipboard`
- Progress bar updates on every `input` event via event delegation on `document`

### JavaScript Functions

| Function | Sync | Description |
|---|---|---|
| `checkPw()` | sync | Validates `#pwInput` against hardcoded password; hides `#passwordGate` on success |
| `switchRespondent(who)` | sync | Shows/hides `#form-student` or `#form-father`; toggles `.active` on selector buttons; calls `updateProgress()` |
| `updateProgress()` | sync | Counts filled `<textarea>` elements in the active form; sets `#formProgress` width as a percentage |
| `collectAnswers()` | sync | Walks `<textarea>` elements in the active form; returns array of `{respondent, question, answer}` objects (blanks skipped) |
| `submitForm()` | **async** | Validates respondent + answers; POSTs payload to `SCRIPT_URL`; shows `#successOverlay` on success; clipboard fallback on error |
| `closeOverlay()` | sync | Removes `.show` class from `#successOverlay` |

### Key Element IDs

| ID | Purpose |
|---|---|
| `passwordGate` | Full-screen password overlay (`display:none` after auth) |
| `pwInput` | Password `<input>` — value compared in `checkPw()` |
| `pwError` | Error message shown on wrong password |
| `formProgress` | Progress bar `<div>` — `width` set as `%` string |
| `btn-student` | Respondent selector button — toggled `.active` class |
| `btn-father` | Respondent selector button — toggled `.active` class |
| `form-prompt` | "回答者を選んでください" placeholder (hidden after selection) |
| `form-student` | Entire 俊希 form section (`display:none` until selected) |
| `form-father` | Entire 父 form section (`display:none` until selected) |
| `form-submit-area` | Submit button wrapper (`display:none` until respondent selected) |
| `submitBtn` | Submit `<button>` — disabled during fetch |
| `successOverlay` | Full-screen success overlay — shown by adding `.show` class |

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

### Form Structure

**俊希フォーム** (`#form-student`) — 11 questions in 4 groups:

| Group | Questions |
|---|---|
| 勉強の現状について | Q1 現在の教材・勉強方法, Q2 過去問の進捗, Q3 しんどい場面 |
| 赤本・過去問の使い方について | Q4 復習方法, Q5 出題傾向への気づき |
| 今回のアイデアについて | Q6 欲しい機能・不要な機能, Q7 効率化したいこと, Q8 出力形式の希望（選択肢あり）, Q9 古典・漢文文献の活用方針 |
| 使っているツール・環境について | Q10 使用デバイス, Q11 使用アプリ・サービス |

**父フォーム** (`#form-father`) — 11 questions in 4 groups:

| Group | Questions |
|---|---|
| メモの背景・意図について | Q1 AI解析を思いついたきっかけ, Q2 優先度設定の理由, Q3 出典文献を知りたい狙い, Q4 他に効率化したいこと |
| 俊希の勉強の様子について | Q5 非効率に見える場面, Q6 苦手科目・分野, Q7 得意科目・分野 |
| 学習計画・方針について | Q8 共テ vs 二次の時間配分, Q9 塾・予備校の有無, Q10 模試成績の現状 |
| 今回の取り組みについて | Q11 期待・懸念 |

Each question uses a `<textarea data-q="Q番号">` — the `data-q` attribute is read by `collectAnswers()` to build the question label in the payload.

## Development Workflow

There is no build step. To make changes:

1. Edit `index.html` directly
2. Open it in a browser to verify (no server needed — `file://` works)
3. Commit and push

For layout/responsive testing, resize the browser window below 768 px or use browser DevTools device emulation.

For print/PDF testing, use browser print preview (`Ctrl+P` / `Cmd+P`).

## Common Modification Recipes

### Add a new slide
1. Pick an insertion point between two existing `<!-- ===== SLIDE N: ... ===== -->` comment markers.
2. Renumber the comment markers of subsequent slides so the sequence stays contiguous.
3. Use `<div class="slide slide-dark">` or `slide-dark-2` to alternate the background.
4. Inside, follow the existing pattern: `.slide-label` → `.section-head` (with `.highlight-inline` span) → `.section-sub` → content components → `.ai-stamp` at the bottom.
5. Update the **Slide Map** table in this file.

### Add a new question to a form
1. Locate the appropriate `.form-q-group` (or add a new group with `.form-q-group-title`).
2. Append a `.form-q-block` containing a `.form-q-label` (with `<span class="qn">Q番号.</span>` prefix) and a `<textarea data-q="Q番号" placeholder="...">`.
3. The `data-q` attribute is what gets serialized — make it unique within that form (`Q12`, `Q13`, ...).
4. `collectAnswers()` and `updateProgress()` pick the new textarea up automatically; no JS changes needed.
5. Update the **Form Structure** tables in this file.

### Change the password
- Edit the literal in `checkPw()` (around line 553–559). Remember this is **client-side only** — never use it as a real secret.

### Change the Google Apps Script endpoint
- Edit the `SCRIPT_URL` constant in the second `<script>` block. The endpoint must accept a POST with `Content-Type: text/plain` (used because `mode: 'no-cors'` strips JSON content-type).

### Tweak colors / theme
- Edit the CSS custom properties on `:root` at the top of the `<style>` block. Most components reference tokens rather than hardcoded colors, so a single change cascades.
- If you add a new design token, document it in the **CSS Design Tokens** table above.

### Add a respondent (beyond 俊希 / 父)
- Not currently supported by `switchRespondent(who)` — it only branches on `'student'` / `'father'`. Adding a third respondent requires updating `switchRespondent()`, `updateProgress()`, `collectAnswers()`, the button row, and the form container `display` toggle.

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
