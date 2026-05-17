---
name: grill-me-visual
description: Generate a single-file HTML questionnaire with visual previews (UI mockups, CSS animations, Mermaid diagrams, font samples, content rendering rules) when the user needs to make multiple visually-shaped decisions — layout, motion, typography, diagram shape, content rendering rules, etc. Fire PROACTIVELY whenever you're about to ask the user 3+ choices in chat that have visual trade-offs they really need to *see* to decide well — don't wait for them to ask. 
---

# grill-me-visual

Emits a **single-file HTML questionnaire** the user opens in a browser. Each question is a card in an accordion with rich visual previews. The user answers in the browser, clicks **Copy**, then pastes the markdown summary back into chat.

## Scope

Use this skill only when the decision space is visually-shaped — UI layout, motion, typography, color, diagram shape, or content rendering rules (how code / markdown / icons / math / tables get displayed). If the decisions are text only with no visual angle, ask them in plain chat instead — the HTML overhead isn't worth it.

## Before invoking

If you're mid-conversation and notice you're about to ask the user a series of visual choices — pause and invoke this skill instead of asking in chat. Threshold: 3+ visual decisions in the immediate next turn, or 2 decisions where seeing the options side-by-side would meaningfully change the answer.

---

## Picking a template

Two templates ship with this skill — pick by user language:

| File | Use when |
|---|---|
| `template.en.html` | The user writes English, or asks the questions in English. All chrome strings and the demo questions are English. |
| `template.zh.html` | The user writes Chinese, or asks the questions in Chinese. All chrome strings and the demo questions are Chinese (Simplified). |

Both templates share identical CSS / JS / accordion / clipboard / Mermaid wiring — only the textual content differs. Mixing languages within one file works but reads poorly; pick one and stick to it for the round.

For other languages, copy the closest fit (probably `template.en.html`) and translate the chrome strings — see "Chrome strings reference" below.

---

## Workflow per round

1. **Identify the topic** — what design space are we exploring this round? (e.g. "AI chat interface", "marketing landing page", "dashboard information hierarchy", "Slack-bot message styling")
2. **Choose 4–6 questions** — each must be a real decision with genuine stakes. No filler / "what's your favorite color" questions.
3. **Pick a preview pattern per question** (see pattern catalog below). See Content rules for the all-or-none rule.
4. **Create + smoke-test the HTML file** by copying the right template (see authoring flow below).
5. **Hand the file path to the user** and wait for their answers.
6. **When they paste the markdown summary back**, parse the answers. Decide whether to:
   - Run another round (new questions informed by these answers)
   - Move on to building based on the decisions
7. **End of session: ask the user about decision capture** — recommend ADR / CONTEXT.md / PRD if any decisions deserve long-term storage. **Do not auto-write** the doc; let the user decide.

---

## Authoring flow — how to fill the template

The template is a single HTML file (~1900 lines, ~30k tokens — **larger than the default Read window**). Don't slurp the whole file. Use these patterns:

- `Bash grep -n "Qn START"` to locate a card's line number, then `Read` with `offset` + `limit` (~80 lines per card)
- The `<!-- Qn START · PATTERN: ... -->` comment header tells you what preview pattern each demo card uses — match yours to one of them
- Each card is fully self-contained between its START and END markers; you can `Edit` that block in one call without reading the rest of the file

**Do not rewrite the template** — copy it and edit only the question content.

```
1. Bash:  cp <skill-dir>/template.<en|zh>.html <output-path>
          # <skill-dir> = the directory this SKILL.md lives in. Resolve from the
          # path the harness gave you when loading the skill.
          # <output-path> = either a project-local path like
          # "grill-{topic-slug}-round-{N}.html" (preferred for multi-round, ask before committing)
          # or $(mktemp -t grill-XXXXXX.html) for one-shot throwaway use.
2. Edit:  replace `const TOPIC = "..."` with this round's topic
3. Edit:  replace `const ROUND = ...` if this is round N > 1
4. Edit:  replace masthead `<span class="topic">...</span>` text to match TOPIC
5. Edit:  replace the `<p class="lede">...</p>` with a roadmap-style POV sentence
6. Edit × N cards:  replace each card body between its
          `<!-- ─────────── Qn START · PATTERN: ... ───────────` and
          `<!-- ─────────── Qn END ─────────── -->` markers.
          Card IDs go q1, q2, … qN; the markdown output groups answers by this id.
7. Smoke-test in a browser BEFORE handing to the user.  `open <path>`,
          click through every option in every card, hit Copy, sanity-check
          the markdown output. Common breakage to catch:
          - Mermaid render failure (syntax violates the hard constraints below)
          - data-id mismatch (card numbering vs `qN` slug)
          - Missing data-rec="1" on the recommended option
          - Fonts silently falling back to system (font not loaded in <head>)
8. Hand off.  Give the user the absolute path. On macOS, `open <path>` opens
          their default browser directly. For phone/tablet review, serve via
          your project's dev server on LAN.
```

**Do not touch:**
- `<head>` (CSS, Google Fonts, viewport meta, theme-color, copy fallback are pre-wired)
- The `<script>` block at the bottom (Mermaid init, accordion logic, copy/reset, dirty-flag tracking, mobile-safe clipboard fallback)
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
- 3 `<button class="opt opt-rich" data-value="slug" [data-rec="1"] data-label="A. …">` options with `.opt-preview` + `.pros-cons` + `.opt-label`
- 1 `<div class="opt opt-input">` for free-text "Other…" (always last)

Tags inside `<span class="opt-tags">…</span>`: any combination of `<span class="opt-tag tag-rec">Recommended</span>` and `<span class="opt-tag tag-current">Current</span>`.

---

## Preview pattern catalog

Pick ONE pattern per question. All preset options within one question MUST use the same pattern.

**Live examples**: open `template.en.html` or `template.zh.html` in a browser — each of the 6 demo cards (Q1…Q6) showcases one of the patterns below in the same order. Read the source of the demo card matching the pattern you want and copy its structure.

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
4. **All-or-none preview** — within one question, either every preset option has a `.opt-preview` mockup, or none does.
5. **Recommended option gets `data-rec="1"` + `<span class="opt-tag tag-rec">{Recommended}</span>`** inside `.opt-tags`. The `data-rec="1"` attribute is what triggers the `(recommended)` suffix in the markdown output — dropping it silently degrades the answer summary. Translate the tag label to match the template language.
6. **Tags are your call** — agent decides per option whether to mark it `tag-rec`, `tag-current` (user is doing this currently), both, or neither. The visual system supports any combination.
7. **D-option placeholder must be context-specific** — e.g. "Other container style…" not generic "Other…".
8. **Lede uses agent's roadmap POV** — frame it as the agent's spine of decisions ("From X to Y to Z — these N decisions define …"), not as instructions to the user ("Answer these N questions about …").

---

## Chrome strings reference (only when translating to a new language)

Both shipped templates have these strings already translated. If you adapt to a third language, replace these:

| Slot | English | Chinese (Simplified) |
|---|---|---|
| Eyebrow | `Grill · Round N` | `Grill · 第 N 轮` |
| Output panel title | `Answers` | `回答汇总` |
| Copy button | `Copy` | `复制` |
| Copied state | `Copied` | `已复制` |
| Copy failed | `Failed · press ⌘C manually` | `失败 · 请手动 ⌘C` |
| Reset button | `Reset` | `重置` |
| Done banner title | `All decided — bring answers back to chat` | `决策完毕 — 把答案带回对话` |
| Done banner CTA | `Copy answers →` | `复制答案 →` |
| Done banner copied state | `✓ Copied — paste it into the chat now` | `✓ 已复制 — 现在去粘回对话框` |
| `tag-rec` label | `Recommended` | `推荐` |
| `tag-current` label | `Current` | `当前` |
| Warn row goto | `Go fill in` | `去补答` |
| Warn row force | `Copy anyway` | `仍然复制` |
| Markdown heading | `## Grill answers · Round N · {TOPIC}` | `## Grill 回答 · 第 N 轮 · {TOPIC}` |
| Markdown custom | `- Custom: {value}` | `- 自填: {value}` |
| Markdown choice | `- Choice: \`{value}\` — {label}` | `- 选择: \`{value}\` — {label}` |
| Markdown unanswered | `- _(unanswered)_` | `- _(未回答)_` |
| Markdown recommended suffix | ` (recommended)` | `（推荐）` |
| Unanswered warning | `{labels} not answered yet. Copy anyway?` | `{labels} 还没回答，仍要复制吗？` |

