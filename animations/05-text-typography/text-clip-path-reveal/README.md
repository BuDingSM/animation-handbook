# 裁剪揭示文字

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/text-clip-path-reveal/)

## 它是什么
裁剪揭示文字让标题一行一行地被揭开，就像一个隐藏的窗口划过每一行。每个字母从一开始就待在它最终的位置上；只有每行可见的部分在变大。因为文字本身从来不动，它的间距保持得和设计时一模一样，这在逐个给字母做动画时是很难保持的。

## 什么时候用
- 多行展示标题，应该读作一个完整的排版单元
- 标语，排版师精确的间距必须保留
- 摄影或设计作品集，字体质量本身就是品牌的一部分
- 任何逐字错开会打断整句阅读的标题

## 工作原理
每一行都包在一个 `overflow: hidden` 的容器里。文字元素先用一个把它完全藏起来的 `clip-path`，然后过渡到完全可见：

```css
:root {
  --line-dur: 700ms;
  --ease: ease-out;
}

.line-text {
  display: block;
  clip-path: inset(0 100% 0 0);   /* hidden: 100% clipped from right */
  transition: clip-path var(--line-dur) var(--ease);
}

.line-text.revealed {
  clip-path: inset(0 0% 0 0);     /* visible: nothing clipped */
}
```

按行的索引加一个 CSS `transition-delay` 来错开各行，或者在 JavaScript 里用 `setTimeout` 的偏移来加 class：

```js
lines.forEach((span, i) => {
  setTimeout(() => span.classList.add('revealed'), stagger * i);
});
```

**方向变体**用不同的初始 clip-path 值：
```css
/* Left-to-right (default) */
.ltr { clip-path: inset(0 100% 0 0); }

/* Right-to-left */
.rtl { clip-path: inset(0 0 0 100%); }

/* Top-to-bottom */
.ttb { clip-path: inset(0 0 100% 0); }
```

揭示时都变成 `inset(0 0% 0 0)`——一个 `.revealed` class 就能处理所有方向。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 起始方向 | 左 | 每行擦除开始的一边：从左开始符合阅读顺序，从右适合从右到左的文字，从上适合展示标题 |
| 速度 | 普通 | 每行揭开需要的时间：慢速 1100ms，普通 700ms，快速 400ms |
| 行间延迟 | 中 | 下一行开始前的等待：短 100ms，中 180ms，长 300ms；100 到 200ms 读起来像是自然的层叠 |
| 感觉 | 平滑 | 平滑是慢慢停下来，感觉很物理；柔和是进出都缓和；均匀保持一个稳定的速度，感觉很机械 |
| 你的行 | Good design / Is honest / Always. | 标题，每行一行 |

## 生产注意事项
- **没有布局位移**：和逐字错开不同，裁剪揭示对布局零影响——文字始终在最终位置上，只是被藏起来。这意味着没有回流，也没有行断变化中途改变的风险。
- **不需要 `overflow: hidden`**：和需要溢出裁剪的 `translate` 揭示不同，`clip-path` 是自包含的。不需要操作父级的溢出。
- **复杂形状用 SVG 裁剪路径**：CSS 的 `inset()` 和 `circle()` 覆盖大多数情况。对于有形状的揭示（斜向擦除、不规则遮罩），用 SVG 的 `<clipPath>` 元素。原理是一样的，只是遮罩形状不同。
- **GSAP**：`gsap.from(lines, { clipPath: 'inset(0 100% 0 0)', duration: 0.7, stagger: 0.18, ease: 'power2.out' })`。干净的一行，不用切换 class。
- **Intersection Observer 搭配**：生产中用 `IntersectionObserver` 在标题滚动进入视野时触发揭示。演示在加载/重播时触发；和 [滚动显现](../../01-scroll-based/reveal-on-scroll/) 模式结合，用于滚动触发的变体。

## 另见
- [剪裁路径揭示](../../02-entrance-and-exit/clip-path-reveal/) — 一个形状揭开任何元素
- [幕布揭示](../../02-entrance-and-exit/curtain-reveal/) — 一块彩色面板盖住它，然后滑走
- [入场退场排版](../enter-exit-typography/) — 每句短语进来、停留、再离开
