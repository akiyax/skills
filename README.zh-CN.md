# grill-me-visual

[English →](./README.md)

> 用看的来做视觉设计决策——在浏览器里从渲染好的选项里挑,而不是在聊天里干描述。

[▸ 打开 live demo](https://akiyax.github.io/skills/grill-me-visual/template.zh.html)

<img src=".assets/grill-me-visual/screenshot.zh.png" alt="grill-me-visual hero" width="400">

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
