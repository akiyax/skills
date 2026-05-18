# grill-me-visual

<p align="center"><strong>English</strong> · <a href="./README.zh-CN.md">中文</a></p>

> *"No-one knows exactly what they want."*  
> — Thomas & Hunt, *The Pragmatic Programmer*
>
> Especially before they've seen it.

<table>
<tr>
<th width="50%" align="left"><code>grill-me</code> — in chat</th>
<th width="50%" align="left"><code>grill-me-visual</code> — in browser &nbsp; <a href="https://akiyax.github.io/skills/skills/grill-me-visual/template.en.html">🚀 Try Now</a></th>
</tr>
<tr>
<td valign="top">

**Q.** How should assistant replies be laid out?

How assistant replies appear on screen — sets the overall "conversation" vs "document" feel, and shapes long-reply readability, information density, and scroll rhythm.

**A. Document style** — *recommended*

**B. Bubble style**

**C. Card style**

*Recommended:* **Document style** — long replies read comfortably; density is high; unless you're building an IM-style social product (Discord, Slack), don't force bubbles.

</td>
<td valign="top">

<img src=".assets/grill-me-visual/screenshot.en.png" alt="grill-me-visual" width="100%">

</td>
</tr>
</table>

## Install

**A. skills CLI** *(recommended)*

Auto-detects which agent you're running (Claude Code, Cursor, Codex, Gemini CLI, OpenCode, etc.) and installs to the right location.

```bash
npx skills add akiyax/skills --skill grill-me-visual -g -y
```

**B. Claude Code plugin marketplace**

```
/plugin marketplace add akiyax/skills
/plugin install grill-me-visual@akiyax-skills
```


## About

[`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) is a great skill. But if you're using [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) or [`brainstorm`](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md), you might run into:

- You can't tell what the agent is actually asking
- You can't tell what each option really means

This is especially true for UI/UX design, process, or architecture decisions — plain text and ASCII diagrams aren't expressive enough. So why not put the questions in HTML?

`grill-me-visual` ships with HTML templates and 6 built-in question types:

- **UI Layout** — layout / container / render-style comparison via inline HTML/CSS
- **Animation** — live `@keyframes` motion demo
- **Mermaid** — flowchart / sequence / state / ER / class diagrams
- **Font** — typography sample comparison
- **Code** — same default rendering, different content
- **Text-only** — fallback for decisions with no visual angle

Your agent writes the question content first, then picks a template per question and creates an HTML file for you to decide in the browser.

When you're done, paste the Q&A summary back into chat to continue.

> [!WARNING]
> **Not a lightweight skill.** Despite token-saving measures, the agent still typically takes a few minutes to assemble the HTML file. Weigh whether the decisions are worth the time/token cost before invoking.
