# grill-me-visual

<p align="center"><a href="./README.md">English</a> · <strong>中文</strong></p>

> 用看的来做视觉设计决策——在浏览器里从渲染好的选项里挑,而不是在聊天里干描述。

<table>
<tr>
<th width="50%" align="left"><code>grill-me</code> — 聊天里</th>
<th width="50%" align="left"><code>grill-me-visual</code> — 浏览器里 &nbsp; <a href="https://akiyax.github.io/skills/grill-me-visual/template.zh.html">🚀 试一试</a></th>
</tr>
<tr>
<td valign="top">

**Q.** 回复消息的整体排版语言?

助手的回复怎么呈现给用户——决定整体的"对话感"vs"文档感",影响长回复的阅读舒适度、信息密度、滚动节奏。

**A. Document 文档风** — *推荐*

**B. Bubble 聊天风**

**C. Card 卡片风**

*推荐:* **Document 文档风** — 长回复阅读舒适、信息密度高;除非专做对话社交产品(IM、Discord 类),别强加 bubble。

</td>
<td valign="top">

<img src=".assets/grill-me-visual/screenshot.zh.png" alt="grill-me-visual" width="100%">

</td>
</tr>
</table>

## 这玩意儿干啥的

有些设计决策在文字里讲不清。*"bubble 还是 document 排版?"、"指数退避还是降级备用?"、"wordmark 用 Source Serif 还是 Playfair?"* —— 你能在聊天里把选项描述出来,但只有把它们并排放在一起看,你才真的知道想要哪个。

`grill-me-visual` 是个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill,在 Claude 准备一口气问你 3+ 个视觉决策时触发。它不在聊天里问,而是**生成一个单文件 HTML 问卷** —— 用任何浏览器打开,每题选项并排预览(UI mockup / CSS 动画 / Mermaid 图 / 字体样本 / 代码片段),推荐项有标记,点选完点 Copy,把 markdown 总结贴回聊天。零构建、零安装,就一个文件。

## 安装

```bash
npx skills add akiyax/skills --skill grill-me-visual -g -y
```

自带英文和中文模板 —— Claude 按你在聊天里用的语言自动挑。

## 背景

灵感来自 [`mattpocock/skills` 的 `grill-me`](https://github.com/mattpocock/skills) —— 同一种"穷追不舍"的 grilling 哲学,只是面向那些 3 句文字问题撑不起来、要 3 张缩略图才说得清的视觉决策。

## License

MIT
