---
name: grill-me-visual
description: Generate a single-file HTML questionnaire with side-by-side visual previews — UI mockups, CSS animations, Mermaid diagrams, font/typography samples, layout variations, code rendering comparisons — so the user picks from rendered options in the browser instead of describing choices in chat. Use for any visually-shaped design decision: layout, motion, typography, color, diagram shape, icon style, content rendering rules ("show me, don't tell me"). Useful for design reviews, design-system exploration, UI prototype direction, and surfacing ADR-worthy decisions. Ships with English and Chinese templates. Fire PROACTIVELY whenever you're about to ask the user 3+ visual-trade-off questions in chat, OR 2 questions where seeing options side-by-side would meaningfully change the answer — don't wait for them to ask.
---

# grill-me-visual

Emits a **single-file HTML questionnaire** the user opens in a browser. Each question is a card in an accordion with rich visual previews. The user answers in the browser, clicks **Copy**, then pastes the markdown summary back into chat.

## Scope

Use this skill only when the decision space is visually-shaped — UI layout, motion, typography, color, diagram shape, or content rendering rules (how code / markdown / icons / math / tables get displayed). If the decisions are text only with no visual angle, ask them in plain chat instead — the HTML overhead isn't worth it.

---

## Picking a template

Two templates ship with this skill — pick by user language:

| File | Use when |
|---|---|
| `template.en.html` | The user writes English, or asks the questions in English. All chrome strings and the demo questions are English. |
| `template.zh.html` | The user writes Chinese, or asks the questions in Chinese. All chrome strings and the demo questions are Chinese (Simplified). |

Both templates share identical CSS / JS / accordion / clipboard / Mermaid wiring — only the textual content differs. Mixing languages within one file works but reads poorly; pick one and stick to it for the round.

For other languages, copy the closest fit (probably `template.en.html`) and translate the chrome strings — see "Translating to a third language" below.

---

## Workflow per round

1. **Identify the topic** — what design space are we exploring this round? (e.g. "AI chat interface", "marketing landing page", "dashboard information hierarchy", "Slack-bot message styling")
2. **Output the question list to chat — before opening the template.** Include only decisions where the user could plausibly pick any option. Reading the template first will anchor you into filler questions and forced patterns.
3. **Pick a preview pattern per question** (see pattern catalog below). Pattern reuse is fine — a layout-heavy round may use UI mockup 3–4× in a row. If a question has no genuine visual angle, switch it to text-only (pattern #6) or ask it in chat.
4. **Create + smoke-test the HTML file** by copying the right template (see authoring flow below).
5. **Hand the file path to the user** and wait for their answers.
6. **When they paste the markdown summary back**, parse the answers. Drill until every visually-shaped decision is resolved — don't stop at one round just because the user answered everything:
   - If any answer reveals a new unresolved dependency (e.g. "Card layout" chosen → now spacing rhythm and border style matter), run another round targeting that branch
   - If answers conflict or raise new trade-offs, surface them in the next round before building
   - Only move on to building when no open visual decision remains
7. **End of session: ask the user about decision capture** — recommend ADR / CONTEXT.md / PRD if any decisions deserve long-term storage. **Do not auto-write** the doc; let the user decide.

---

## Authoring flow — how to fill the template

The template is a single HTML file (~1900 lines, ~75 KB, roughly 20–25k tokens — **larger than the default Read window**). Don't slurp the whole file. Use these patterns:

- `Bash grep -n "PATTERN:"` lists what pattern each demo card uses. Delete the cards for patterns you don't need (whole `<!-- QN START -->`…`<!-- QN END -->` block) sight-unseen — no Read.
- For the survivors, `grep -n "Qn START"` to locate, then `Read` with `offset` + `limit` (~80 lines per card)
- Each card is fully self-contained between its START and END markers; you can `Edit` that block in one call without reading the rest of the file

**Do not rewrite the template** — copy it and edit only the question content.

```
1. Bash:  mkdir -p .grill-me-visual && cp <skill-dir>/template.<en|zh>.html <output-path>
          # <skill-dir> = the directory this SKILL.md lives in. Resolve from the
          # path the harness gave you when loading the skill.
          # <output-path> = .grill-me-visual/grill-{topic-slug}-round-{N}.html
          # — hidden top-level folder in the user's CWD (dot prefix keeps it out
          # of `ls` output and avoids colliding with the skill's own source dir
          # when running inside this repo). Decisions belong to the project
          # they're about; multi-round files cluster together; the user can
          # commit them or gitignore them at their own discretion.
          # Use a relative path with forward slashes so it works cross-platform
          # (macOS, Linux, WSL, Git Bash, PowerShell, cmd). Avoid `/tmp`
          # (missing on native Windows) and `mktemp -t ...` (semantics differ
          # between macOS and Linux).
2. Bash:  grep -n "PATTERN:" <file>  # which Qn uses which pattern
          Delete whole QN blocks for any pattern you're NOT using (markers and
          all) — no Read needed. Renumber survivors' `data-id` sequential q1…qN.
3. Edit:  replace `const TOPIC = "..."` with this round's topic
4. Edit:  replace `const ROUND = ...` if this is round N > 1
5. Edit:  replace masthead `<span class="topic">...</span>` text to match TOPIC
6. Edit:  replace the `<p class="lede">...</p>` with a roadmap-style POV sentence
7. Edit × N cards:  replace each surviving card body between its
          `<!-- ─────────── Qn START · PATTERN: ... ───────────` and
          `<!-- ─────────── Qn END ─────────── -->` markers.
          Card IDs go q1, q2, … qN; the markdown output groups answers by this id.
8. Sanity check BEFORE handing off. If you have browser access (Chrome MCP,
          Playwright, headless browser, etc.), open the file and verify Mermaid
          renders, fonts load, and layout holds. Always also grep-check structure:
          - `grep -c 'data-id="q' <file>` matches your intended card count (and IDs are q1…qN sequential)
          - `grep -c '<button[^>]*data-rec="1"' <file>` == card count — every question has exactly one recommended option
          - `data-value="..."` slugs are unique within each card
          Without browser access, also eyeball-check from source:
          - Mermaid `<pre class="mermaid">` blocks against the hard constraints (no `:`, no `"`, no `<`/`>`, basic shapes only)
          - Every `font-family` referenced is loaded via the Google Fonts <link> in <head>
9. Hand off.  Give the user the absolute path and ask them to open it in a browser
          to verify rendering. On macOS that's `open <path>`; for phone/tablet
          review serve via your project's dev server on LAN. If they spot a
          Mermaid render failure / font fallback / layout issue, fix and re-hand.
```

**Do not touch:**
- `<head>` (CSS, Google Fonts, viewport meta, theme-color, copy fallback are pre-wired). Exception: when translating, swap CDN URLs if needed.
- The `<script>` block at the bottom (Mermaid init, accordion logic, copy/reset, dirty-flag tracking, mobile-safe clipboard fallback, tag-rec auto-inject). Exception: when translating, edit the `LANG = { ... }` object at the top of `<script>` and nothing else — see "Translating to a third language" below.
- Styles for `.card` / `.opt` / `.opt-tags` / `.chevron` / scrollbars / dark mode
- Theme variables (`--paper`, `--ink`, `--clay`, `--moss`, etc.)

If you find yourself wanting to change CSS/JS, **stop and ask the user** — the template already handles mobile-safe copy, font fallback (incl. Chinese), dark-mode contrast, Mermaid theming, and small-screen layout.

---

## Card schema — what each card needs

Each card replaces the block between `<!-- Qn START -->` and `<!-- Qn END -->`. Open Q1 in the template for the canonical structure — copy it and substitute your content. Required slots (in order):

- `data-id="qN"` on the wrapping `<div class="card">` (must match position in 1–N sequence)
- `<span class="qnum">Qn.</span>` and `<span class="title-text">` in `.card-header`
- `<p class="q-desc">` — what / why (Setting → impact). No option preview here.
- `<p class="rec-hint"><b>{name}</b> — {one-line why}</p>` — recommendation reasoning
- N `<button class="opt opt-rich" data-value="slug" [data-rec="1"] data-label="A. …">` preset options with `.opt-preview` + `.pros-cons` + `.opt-label`. Letters A, B, C, … sequential; exactly one option has `data-rec="1"`. Use as many as the decision genuinely has — if you can't find 2, ask in chat instead; if you'd need 5+, consider splitting into two questions.
- 1 `<div class="opt opt-input">` for free-text "Other…" — always last, letter following the last preset.

Tags: set `<span class="opt-tags"><span class="opt-tag tag-current">Current</span></span>` on the option the user is using today (if any). The `Recommended` tag is auto-injected from `data-rec="1"` — don't add it by hand. To translate the recommended label, edit `LANG.tagRec` only.

---

## Preview pattern catalog

Pick ONE pattern per question. All preset options within one question MUST use the same pattern.

| Pattern | Demo card | Use when |
|---|---|---|
| #1 UI mockup | Q1 in template | "how it should look" — layout / container / render style |
| #2 CSS animation | Q2 in template | motion — loading, streaming, transition |
| #3 Mermaid | Q3 in template | shape of a flow / state / sequence |
| #4 Font | Q4 in template | typography — body, wordmark, display |
| #5 Code snippet | Q5 in template | content comparison — same default rendering, different content |
| #6 Text-only | Q6 in template | no genuine visual angle (last resort) |

Open `template.en.html` or `template.zh.html` and read the matching demo card as the canonical structure to copy from. If none of these 6 fit, invent your own pattern — the preview HTML/CSS goes inside `.opt-preview`; keep the rest of the card structure unchanged. Add a `<!-- PATTERN: ... -->` comment so the next round can spot it.

### 1. UI mockup (inline HTML/CSS rendering of a tiny real UI)
Use this when the decision is "how the UI should look" — different layouts, container styles, render styles for the same content. Use real placeholder copy at microscale (~60–70% of real proportions), not gray-bar wireframes (those look like skeleton loaders).

### 2. CSS animation live demo (real `@keyframes`)
Use this when the decision is about motion — what to show while loading / streaming / transitioning. Animations render inline so the user actually sees them move.

The template predefines reusable keyframes in its `<style>` block (`demo-cursor-blink`, `demo-dots`, `demo-fade-token`). Reuse those for cursor / dots / fade patterns rather than defining new ones; only add new `@keyframes` if you need a different motion.

### 3. Mermaid flowchart / sequenceDiagram / stateDiagram
Use this for "process / state / control flow" decisions where the comparison is *shape of the flow*. Drop a `<pre class="mermaid">…</pre>` inside each option's `.opt-preview`.

**Hard constraints (the template's Mermaid renderer assumes basic shapes only):**
- No double quotes inside labels — write `[Send]` not `["Send"]`
- No colons in node labels — write `A[Send · message]` or `A[Send — message]`, not `A[Send: message]`
- No `&lt;` / `<` / `>` in labels — they don't decode. Reword the condition: write `D{Can retry?}` not `D{Retries < N?}`
- Only basic shapes: `id[text]` `id{text}`. No stadium `([..])`, hexagon `{{..}}`, parallelogram `>..]`, asymmetric `>..]`
- Use `flowchart TD` / `LR`, `sequenceDiagram`, `stateDiagram-v2`, `erDiagram`, or `classDiagram` — the themed CSS overrides target these
- Do NOT add custom theme directives (`%%{init: ...}%%`), `classDef`, or inline `style` statements — they collide with the brand palette overrides; let the template's defaults render

### 4. Font comparison (same text rendered in each font candidate)
Use this for typography decisions — body system, wordmark, display headline, etc.

**Important**: if any candidate is a Latin-only font (Source Serif 4, Inter Tight, Geist, etc.), the differences only show on Latin characters — non-Latin scripts (Chinese, Japanese, etc.) fall back to the system font on every option. If the user is comparing for a CJK product and you want the difference to read in mixed Latin + CJK, use a mixed sample explicitly.

**Mandatory: load every candidate font.** Referencing `font-family: 'Geist'` in an inline `style` without loading it = silent fallback to system, and the comparison is broken.

The template's `<head>` already loads: Source Serif 4 (serif display/body), Manrope (sans), Noto Sans SC + Noto Serif SC (Chinese fallbacks). To add more, append `&family=<Name>:<spec>` to the Google Fonts `<link>` URL — copy the spec from fonts.google.com (spaces become `+`).

### 5. Code snippet comparison (same default rendering, different content)
Use this when the decision is about the *content* of code — competing data shapes, algorithms, API signatures, prompt formats, config schemas — and each option is a small snippet of code with the same look.

The template provides a `.code-snippet` component for the default rendering — header with language tag, then a syntax-highlighted `<pre>` body. Drop it inside each option's `.opt-preview`:

```html
<div class="code-snippet">
  <div class="code-head"><span class="code-lang">typescript</span></div>
  <pre><span class="tok-kw">const</span> x = <span class="tok-num">1</span>;</pre>
</div>
```

Optional highlight token classes: `.tok-kw` (keyword), `.tok-id` (identifier), `.tok-str` (string), `.tok-num` (number), `.tok-cmt` (comment). Plain text inside `<pre>` is fine — highlighting is decorative.

> If the decision is instead about *how* content should be rendered (line numbers vs syntax highlight for code, table style for tabular data, emoji vs SVG for icons, KaTeX vs MathJax for math, etc.), that's a **UI mockup question** (pattern #1), not a content-snippet question — the "UI" you're comparing is the rendering component itself.

### 6. Text-only decision
For decisions with no visual angle (e.g. "should the AI greet the user proactively?"), use `opt-rich` but put only `<ul class="pros-cons">` inside `.opt-preview` — no visual mockup. The "all-or-none preview" rule means ALL options in this question must be text-only.

For genuinely open-ended questions where some options have "no UI" (e.g. silent greeting = blank screen), use a plain `.opt` with no `opt-rich`:

```html
<button type="button" class="opt" data-value="none" data-label="A. None">
  <span class="key">A.</span>None (stay with current)
</button>
```

---

## Content rules (apply to every question)

1. **q-desc structure: setting → impact.** Explain when the decision applies and what it affects. Do **not** preview the options inside q-desc.
2. **rec-hint is the cross-option comparison**, one sentence framing why the recommended option wins *over the alternatives*. `<b>{option name}</b> — {reason it beats the other choices}`. Don't just restate the recommended option's pros — those go in its `pros-cons` list. rec-hint answers "why this one and not the others"; pros-cons answers "what are the trade-offs of this specific option in isolation".
3. **Every preset option needs `<ul class="pros-cons">`** with 2–3 `<li class="pro">` and 1–2 `<li class="con">`. Pros first, then cons. Exception: pure-affordance options like "None" in nullable questions.
4. **All-or-none preview** — within one question, either every preset option has a `.opt-preview` mockup, or none does. Pattern #6 (text-only) is the all-none case: every option's `.opt-preview` holds only pros/cons, no mockup. Don't mix mocked options with plain `.opt` (no `opt-rich`) options in the same question.
5. **Recommended option gets `data-rec="1"`** — that single attribute drives both the `(recommended)` suffix in the markdown output *and* the auto-injected `Recommended` tag in the UI. Don't write the tag span by hand.
6. **`tag-current` is manual, `tag-rec` is automatic.** Add `<span class="opt-tag tag-current">Current</span>` inside `<span class="opt-tags">…</span>` when the user is doing this thing today. The recommended tag appears automatically wherever `data-rec="1"` is set — see rule 5.
7. **D-option placeholder must be context-specific** — e.g. "Other container style…" not generic "Other…".
8. **Lede uses agent's roadmap POV** — frame it as the agent's spine of decisions ("From X to Y to Z — these N decisions define …"), not as instructions to the user ("Answer these N questions about …").

---

## Translating to a third language

The two shipped templates are kept structurally identical — their `<script>` bodies are byte-identical after the `LANG` object — and differ in exactly three places:

1. **The `LANG = { ... }` object** at the top of `<script>` — all strings that JS emits dynamically (markdown output, button states, warning text, the `Recommended` tag label).
2. **Inline HTML chrome strings** — masthead eyebrow / topic, lede, output-panel title, button labels (Copy / Reset / warn-row goto+force), done-banner title + CTA, the `tag-current` label, every demo card's body.
3. **`<head>` CDN URLs** — Google Fonts and Mermaid. Use China-accessible mirrors (`fonts.googleapis.cn`, `cdn.bootcdn.net`) when the audience is in China.

To add a new language: copy whichever template is closest, swap (1)+(2)+(3), and `diff` the JS body against the original to confirm nothing else drifted. Run `diff <(awk '/^const dark = matchMedia/,/^<\/script>/' template.en.html) <(awk '/^const dark = matchMedia/,/^<\/script>/' template.NEW.html)` — it must be empty.

