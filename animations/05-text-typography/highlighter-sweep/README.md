# 荧光笔扫过

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/highlighter-sweep/)

## 它是什么
荧光笔扫过像用笔一样标记句子里的关键词。一条记号笔颜色的带子从左到右在每个关键短语后面长出来，按阅读顺序一个短语接一个短语，让眼睛从一个想法走到下一个。文字本身从来不动；只有它后面的颜色被画出来。

## 什么时候用
- 引用和推荐，标记读者应该记住的那一行
- 落地页标题，一个短语承载着承诺的地方
- 摘要、学习笔记和引导提示，两三个想法最重要
- 随着章节滚动进入视野而被标记的关键点

## 工作原理
每个关键短语是一个 `<mark>`。脚本把它的每个词连同后面的空格包进一个 span，它的 `::before` 就是那条带子：放在词后面（在卡片自己的层叠上下文里用 `z-index: -1`，这样带子永远不会掉到卡片后面），从左边缘缩放到没有，再在段落拿到 `on` class 时长到满宽。因为每个词都有自己的带子，换到第二行的短语会逐行标记；因为后面的空格在 span 里面，一行里带子会连起来。

```css
.paper { position: relative; isolation: isolate; }
.w { position: relative; }
.w::before {
  content: ''; position: absolute; z-index: -1;
  left: -.08em; right: -.08em; top: var(--top); bottom: -.06em;
  background: var(--marker);
  transform: scaleX(0); transform-origin: left center;
  transition: transform var(--t) var(--e) var(--d);
}
.para.on .w::before { transform: scaleX(1); }

@media (prefers-reduced-motion: reduce) {
  .w::before { transform: none; opacity: 0; transition: opacity .4s linear var(--d); }
  .para.on .w::before { opacity: 1; }
}
```

脚本给每个词设定延迟（`--d`）、时长（`--t`）和时间曲线（`--e`）。一个短语用一条平滑曲线 smoothstep，这样笔在一个短语里只加速减速一次，而不是每个词一次。一个词覆盖它短语宽度的 p0 到 p1 部分，就在曲线到达 p0 和 p1 的时间之间运行，它自己的时间曲线正是 smoothstep 的那一段，写成三次贝塞尔曲线，这样笔的速度从一个词到下一个词平滑地延续：

```js
const S = t => t * t * (3 - 2 * t), dS = t => 6 * t * (1 - t);   // the phrase's curve and its slope
const t0 = inv(p0), t1 = inv(p1), k = (t1 - t0) / (p1 - p0);     // inv: where S reaches p0 and p1
w.style.setProperty('--d', start + t0 * D + 'ms');
w.style.setProperty('--t', (t1 - t0) * D + 'ms');
w.style.setProperty('--e', `cubic-bezier(.333, ${dS(t0) * k / 3}, .667, ${1 - dS(t1) * k / 3})`);
```

重播先把所有带子放回没有（过渡先关一会儿），强制重算一次样式，再重新加上 `on`，这样每次播放都从空白的纸上扫过。只有 `transform` 和 `opacity` 在变，所以文字从来不重排。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 荧光笔颜色 | 黄色 | 带子的颜色：黄色、粉色、绿色、蓝色或橙色，都够浅，深色文字仍然容易读 |
| 速度 | 普通 | 记号笔穿过一个短语需要的时间：慢速 1300ms，普通 700ms，快速 400ms，短语之间停 250ms |
| 覆盖范围 | 整词 | 整词会把字母完全盖住：带子从词框顶部下方字体大小的 12% 处开始，这仍然比最高的字母高大约字体大小的八分之一，在 g 和 p 这样的字母尾巴下方一点结束；下半部分从 62% 处开始，大约在大写字母一半高的地方，所以只盖住字母的下半部分，像一条粗下划线 |

## 生产注意事项
- **background-size 版本**：一个 `<mark>` 用 `background: linear-gradient(var(--marker), var(--marker)) no-repeat; background-size: 0% 100%`、`box-decoration-break: clone` 和 `background-size` 上的过渡，不用拆分就能处理换行。这是代码最短的办法，但 `background-size` 每一帧都会重绘；这里的 transform 版本留在合成器上。
- **看到时播放一次**：用 IntersectionObserver 在段落滚动进入视野时开始扫动，不要每次滚动都再跑一次。
- **对比度**：对照每个记号笔颜色检查文字（WCAG 要求 4.5:1）。深色主题用更深、更饱和的记号笔配浅色文字，或者用 `mix-blend-mode` 的半透明记号笔。
- **含义**：`<mark>` 告诉辅助技术这些词是被高亮的，有些屏幕阅读器会说出来。不要只靠颜色来承载文字本身没有传达的含义。
- **拆分**：把空格留在每个词的 span 里，这样带子才能连起来；对不用空格分词的语言，用 `Intl.Segmenter` 拆分。
- **库**：GSAP 的 SplitText 配合错开的 `scaleX` 补间，或者 Framer Motion 的变体配合 `staggerChildren`，能做出同样的东西；Rough Notation 用 SVG 画出手绘的高亮。

## 另见
- [裁剪揭示文字](../text-clip-path-reveal/) — 标题的每一行依次被揭开
- [描边填充](../outline-to-fill/) — 空心字填满颜色，像擦除或淡入
- [滚动文字填充](../../01-scroll-based/text-fill-on-scroll/) — 词随着滚动一个一个亮起来
- [逐词揭示](../../02-entrance-and-exit/word-by-word-reveal/) — 词按阅读节奏一个接一个出现
