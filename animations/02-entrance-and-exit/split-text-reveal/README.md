# 拆字揭示

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/split-text-reveal/)

## 它是什么

拆字揭示把文字拆成块（字、词或行），每块之间隔一小会儿，依次做动画。这个延迟把平淡的淡入淡出变成一道流过文字的级联。它是大多数动画标题入场的基础。

## 什么时候用
- 标题和主视觉入场，字母或词语泛起涟漪般到位
- 逐词揭示，时间安排得像这句话正在被说出来
- 较长文案块的逐行揭示
- 任何单次淡入淡出显得太均匀、想要方向性流动的文字入场

## 工作原理
JavaScript 拆分文字，给每个单元一个 `--i` 序号自定义属性。CSS 再在 `transition-delay` 里把这个序号乘以 `--stagger` 变量，所以单元 0 立刻开始，单元 1 晚一拍，以此类推——不需要每个元素一个计时器：

```css
.unit {
  display: inline-block;
  opacity: 0;
  transition: opacity var(--dur) var(--ease) calc(var(--i) * var(--stagger)),
              transform var(--dur) var(--ease) calc(var(--i) * var(--stagger));
}
[data-anim="fade-up"] .unit { transform: translateY(0.5em); }
.splitting.in .unit { opacity: 1; transform: none; }
```

拆分在创建这些 span 时分配序号：

```js
text.split(/\s+/).filter(Boolean).forEach((word, wi, all) => {
  const w = document.createElement('span');
  w.className = 'unit-word';                 // keeps the word's letters on one line
  [...word].forEach(c => {
    const s = document.createElement('span');
    s.className = 'unit';
    s.style.setProperty('--i', unitIdx++);   // drives its transition-delay
    s.textContent = c;
    w.appendChild(s);
  });
  target.appendChild(w);
  if (wi < all.length - 1) target.appendChild(document.createTextNode(' '));
});
```

每个单元的动画可以通过 `data-anim` 切换（fade-up、fade、scale、rotate）；`lines` 模式还会裁剪每行、把里面的 span 向上滑，把交错和上滑揭示技术结合起来。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 拆成 | 词语 | 字母的级联最细腻，词语的节奏最易读，行是成块揭示 |
| 每块怎么出现 | 淡入上浮 | 每块演的小入场：「淡入上浮」上升半个字号，「淡入」只淡入，「放大」从一半大小开始，「翻转」从 90°、侧面对着你、绕一条竖线转进来；在「行」模式下什么都不淡入淡出：每行都从一条看不见的边缘后面滑入视野，所以「淡入」只显示那个滑动，而「淡入上浮」「放大」「翻转」会在上面加上自己的动作 |
| 块间延迟 | 中等 | 块与块之间的等待：「短」是 10ms，「中等」30ms，「长」100ms；小的值起涟漪，大的值感觉很从容 |
| 速度 | 普通 | 每块要多长时间：「慢」是 800ms，「普通」500ms，「快」300ms |
| 感觉 | 平滑 | 每块用的曲线；「弹回」加一点点落定，「均匀」保持一个稳定的速度 |
| 你的文字 | Design beyond the obvious | 被拆开、被揭示的文字 |

## 生产注意事项
- **总时间 = 单元数 × 交错 + 时长。**长字符串的字符模式乘得很快——40 个字符 × 30ms + 500ms 将近 1.7 秒。限制交错，或者给较长的文案换成词语/行，别让尾巴拖太久。
- **拆分会破坏文字的无障碍性。**屏幕阅读器可能把按字符拆分的文字逐字母读出来，而且拆分会破坏原生的词语换行。在容器上加带完整字符串的 `aria-label`（块上加 `aria-hidden`），并且在调整大小时重新拆分，因为换行会变。
- **有意保留空格。**演示在字符模式里为空格插入真实的文本节点，并在词语 span 之间加尾随空格——没有这些，`inline-block` 单元会把空白折叠起来，词语就连在一起了。
- **减弱动态：**在 `prefers-reduced-motion` 下，演示把每个单元的 transform 都收拢成单纯的 300ms 不透明度淡入淡出，整句话没有级联就出现了。
- **库里的等价写法：**Splitting.js 和 GSAP 的 SplitText 都会帮你做 DOM 拆分（SplitText 还处理 ARIA 和调整大小时重新拆分）；Framer Motion 在父变体上用 `staggerChildren`，子元素是 `motion.span`；Motion One 的 `stagger()` 辅助函数生成每个元素的延迟。

## 另见
- [上滑揭示](../slide-up-reveal/) — 整行从一条看不见的边缘后方升起
- [逐字母出现](../letter-by-letter-stagger/) — 一次一个字母，或者打出来
- [逐词揭示](../word-by-word-reveal/) — 一次一个词，按阅读速度
- [淡入淡出](../fade-in-out/) — 整段文字一次淡入
