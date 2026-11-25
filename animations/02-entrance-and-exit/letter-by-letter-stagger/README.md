# 逐字母出现

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/letter-by-letter-stagger/)

## 它是什么

逐字母出现让每个字符都比前一个晚一会儿动起来，这样一句话会像级联一样组装起来，而不是一下子全出现。演示展示了两个版本：级联版，每个字母都演一个小入场；打字机版，一次揭示一个字母，还可以带一个闪烁光标。

## 什么时候用
- 主视觉标题和落地页标题，文字就是焦点
- 30 个字符以下的短句——效果很夸张，长文案用起来会很烦
- 加载或「组装」状态，想要感觉很从容
- 打字机模式专门用于终端界面、聊天界面和代码演示

## 工作原理
每个字符都包在自己的 `<span class="char">` 里，并给一个等于它序号的自定义属性 `--i`。CSS 把这个序号乘以 `--stagger` 值，算出每个字母的 `transition-delay`，所以第 *n* 个字母在动画触发后 `n × stagger` 毫秒开始：

```js
[...text].forEach((c,i)=>{
  const s=document.createElement('span');
  s.className='char';s.style.setProperty('--i',i);
  s.textContent=c===' '?'\u00a0':c; // a lone plain space would collapse inside its inline-block
  target.appendChild(s);
});
```

```css
[data-mode="cascade"] .char{
  opacity:0;
  transition:opacity var(--dur) var(--ease) calc(var(--i)*var(--stagger)),
             transform var(--dur) var(--ease) calc(var(--i)*var(--stagger));
}
[data-mode="cascade"][data-anim="fade-up"] .char{transform:translateY(0.4em)}
[data-mode="cascade"].done .char{opacity:1;transform:none}
```

加上 `.done` 类会同时把每个字母翻到静止状态；交错的 `transition-delay` 是把可见动作在时间上铺开的东西。打字机模式完全跳过过渡，改用一串 `setTimeout` 调用，依次给每个字符切换 `.visible` 类。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 样式 | 级联 | 「级联」依次给每个字母做动画；「打字机」把它们打出来 |
| 字母间延迟 | 中等 | 字母之间的间隔：「短」是 20ms，「中等」35ms，「长」80ms；越大越慢、越戏剧化 |
| 每个字母怎么出现 | 淡入上浮 | 在「级联」里，每个字母淡入时演的入场：「淡入上浮」上升 0.4 个字号，「淡入」只淡入，「放大」从 30% 大小开始，「翻转」从 90°、侧面对着你、绕一条竖线转进来 |
| 速度 | 普通 | 在「级联」里，每个字母落定要多长时间：「慢」是 650ms，「普通」400ms，「快」250ms |
| 闪烁光标 | 开 | 在「打字机」里，一个闪烁光标领着字母前进 |
| 你的文字 | Hello, world. | 被做动画的短语，最多 60 个字符 |

## 生产注意事项
- **无障碍**：对大多数屏幕阅读器来说，拆分的文字仍然读作分开的 span，但在容器上用 `aria-label` 包住整句话、再用 `aria-hidden` 藏起这些 span，就能保证标签被当作一个字符串读出来。演示在 `prefers-reduced-motion` 下强制所有字母可见、没有过渡。
- **布局成本**：每个 span 都需要 `display:inline-block`，`transform` 才能生效，但这会禁用词语内的正常换行。会换行的标题，先按词语拆分，再拆每个词语里的字母。
- **不要滥用**：逐字母是最细的拆分，也是最快让人烦的。每页只留给一个标题。
- **GSAP**：`SplitText` 加上时间线里的一个 `stagger` 值就是标准实现，它会帮你处理包装 span 的 bookkeeping。
- **Framer Motion**：用一个带 `staggerChildren` 的父 `variants` 对象，把每个字母映射到一个子 `motion.span`。
- **Motion One**：把一个元素数组传给 `animate()`，用 `delay: stagger(0.035)` 辅助函数。

## 另见
- [逐词揭示](../word-by-word-reveal/) — 一次一个词，更适合较长的文字
- [拆字揭示](../split-text-reveal/) — 文字拆成字母、词语或行
- [模糊渐入](../blur-in/) — 淡入时从模糊中变清晰
