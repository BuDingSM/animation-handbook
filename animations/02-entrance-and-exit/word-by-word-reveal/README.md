# 逐词揭示

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/word-by-word-reveal/)

## 它是什么

逐词揭示一次给一段文字里的一个词做动画，而不是一次全部或逐字母。词语是人们阅读的方式，所以每个词大约 80 到 120 毫秒的延迟会带着眼睛走过一个句子：慢到感觉是有意的，快到没人需要等。

## 什么时候用
- 多行正文、引言和段落开场，逐字母会太忙的地方
- 顺序讲故事，想让每个短语在下一个之前落定
- 配合旁白时间的字幕或说明揭示
- 分段揭示，句间停顿分隔不同的想法

## 工作原理
文字按空白拆成 `<span class="word">` 元素，每个带一个 `--i` 序号。CSS 从 `--i × --stagger` 得出 `transition-delay`，在容器上加上 `.in` 类会同时把每个词放到静止状态——延迟把动作扇形铺开：

```js
words.forEach((w,wi)=>{
  const s=document.createElement('span');s.className='word';
  const delay=punctTog.checked?idx*stagger+si*200:idx*stagger;
  s.style.setProperty('--i',delay/stagger);
  s.textContent=w;target.appendChild(s);
  if(wi<words.length-1)target.appendChild(document.createTextNode(' '));
  idx++;
});
```

```css
.word{display:inline-block;opacity:0;
  transition:opacity var(--dur) var(--ease) calc(var(--i)*var(--stagger)),
             transform var(--dur) var(--ease) calc(var(--i)*var(--stagger))}
[data-anim="fade-up"] .word{transform:translateY(0.4em)}
.splitting.in .word{opacity:1;transform:none;filter:none}
```

「句间停顿」开关给每句话的延迟加上 `si × 200ms`，在行之间插入一拍。因为序号是按 `delay/stagger` 存的，不管停顿开不开，CSS 的数学都保持一个 `calc()`。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 词间延迟 | 中等 | 阅读速度控制：「短」是 40ms，「中等」80ms，「长」150ms；不到大约 40ms 时，词语会模糊成一次揭示 |
| 每个词怎么出现 | 淡入上浮 | 每个词淡入时演的入场：「淡入上浮」上升 0.4 个字号，「淡入」只淡入，「放大」从 60% 大小开始，「模糊」从 8px 模糊开始，「翻转」从 90°、侧面对着你、绕一条横线转进来 |
| 速度 | 普通 | 每个词落定要多长时间：「慢」是 700ms，「普通」450ms，「快」270ms |
| 感觉 | 平滑 | 「平滑」让每个词慢慢到位；「弹回」加一点弹跳；「均匀」感觉很机械 |
| 句间停顿 | 关 | 在句子之间加一个额外的停顿 |
| 你的文字 | 三句示例 | 被揭示的段落，每句话一行 |

## 生产注意事项
- **保留空白**：在词语 span 之间留一个真实的文本节点（一个实实在在的空格），浏览器才能正常换行和对齐，单纯的 `margin-right` 间距在不同字体下做不到这一点。
- **运行时间随长度增长**：总时间是 `wordCount × stagger + dur`。一个长段落按 120ms/词算可能要跑好几秒——超过一两句话时，限制词数或交错。
- **减弱动态**：在 `prefers-reduced-motion` 下，演示收拢成一次 300ms 的线性淡入淡出，没有 transform，内容仍然出现，但没有顺序动作。
- **GSAP**：`SplitText({type:"words"})` 加上补间里的 `stagger` 就是直接的等价写法。
- **Framer Motion**：一个带 `staggerChildren:0.08` 的容器变体，加上每个词的子变体，不用手动算序号就能复现。
- **Motion One**：把 `stagger(0.08)` 作为 `delay` 选项传给 `animate()`。

## 另见
- [逐字母出现](../letter-by-letter-stagger/) — 一次一个字母，用于短句
- [拆字揭示](../split-text-reveal/) — 文字拆成字母、词语或行
- [淡入淡出](../fade-in-out/) — 整段文字一次淡入
