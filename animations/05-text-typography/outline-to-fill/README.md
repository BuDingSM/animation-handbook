# 描边填充

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/outline-to-fill/)

## 它是什么
描边填充让文字一开始是空心字，只显示每个字母的描边，然后填满颜色，所以文字看起来像正在被涂上墨。有两种做法，演示都展示了：一种是擦除，从一边像油漆一样填满字母；另一种是淡入，像墨水浸透一样同时填满所有地方。

## 什么时候用
- 海报和编辑风格网站上的主视觉文字，字母形状本身就是视觉
- Logo 揭示和字标动画
- 滚动进入时激活的章节标题
- 任何简单淡入太平淡、完整动态排版又太复杂的标题

## 工作原理
`-webkit-text-stroke` 做出空心描边；`color: transparent` 让填充看不见。两个相同的文字元素叠在一起——描边层始终可见，填充层用裁剪路径动画揭示：

```html
<div class="text-wrap">
  <div class="outline-layer">OUTLINE</div>   <!-- always visible: stroke, no fill -->
  <div class="fill-layer">OUTLINE</div>      <!-- revealed: stroke + fill -->
</div>
```

```css
.text-wrap { position: relative; display: inline-block; }

.outline-layer {
  -webkit-text-stroke: 2px #58a6ff;
  color: transparent;
  position: relative;
  z-index: 2;
}

.fill-layer {
  position: absolute;
  inset: 0;
  -webkit-text-stroke: 2px #58a6ff;
  color: #58a6ff;
  clip-path: inset(100% 0 0 0);   /* fully hidden initially (bottom-up) */
  transition: clip-path 900ms ease-in-out;
  z-index: 1;
}

.fill-layer.filled {
  clip-path: inset(0%);           /* fully revealed */
}
```

在 JavaScript 里触发填充：
```js
document.querySelector('.fill-layer').classList.add('filled');
```

方向变体改变起始的 `clip-path`：
```css
/* Bottom to top (default) */
clip-path: inset(100% 0 0 0);

/* Top to bottom */
clip-path: inset(0 0 100% 0);

/* Left to right */
clip-path: inset(0 100% 0 0);

/* Center outward */
clip-path: inset(50% 0);   /* top+bottom each at 50% = nothing visible */
```

对于透明度交叉淡入的变体（不用裁剪路径）：
```css
.stroke-only { opacity: 1; transition: opacity 900ms ease; }
.fill-only   { opacity: 0; transition: opacity 900ms ease; }

.filled .stroke-only { opacity: 0; }
.filled .fill-only   { opacity: 1; }
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 填充方向 | 自下而上 | 擦除开始的地方：底部、顶部、左侧、右侧或中间；自下而上读起来像在装满一个容器，从左到右像在写字。淡入没有方向 |
| 速度 | 普通 | 填满需要的时间：慢速 1500ms，普通 900ms，快速 550ms |
| 描边粗细 | 中 | 描边有多宽：细是 1px，中 2px，粗 4px；细看起来精确，粗看起来醒目 |
| 感觉 | 柔和 | 柔和是进出都缓和；平滑是慢慢停下来；均匀保持一个稳定的速度 |
| 描边颜色 | 粉色 | 字母描边的颜色 |
| 填充颜色 | 粉色 | 填满字母的颜色 |
| 你的文字 | OUTLINE | 会填满的词，最多 10 个字母，用大写显示 |

## 生产注意事项
- **`-webkit-text-stroke` 是非标准的**：所有现代浏览器（Chrome、Firefox、Safari、Edge）都支持它，但它不在 CSS 规范里。标准替代方案是带扩散的 `text-shadow`——没那么清晰，但兼容性更好。生产中要在目标浏览器和字号下测试描边的渲染。
- **描边分布在填充之外**：`-webkit-text-stroke` 把描边同时分布在字符路径的内外。在粗值下，它会吃掉字母内部空间（counter）。正文字号用细描边（1–3px）；只有非常大的展示文字才用粗描边。
- **双元素开销**：这个手法每个动画词需要两个 DOM 元素。对于许多词或动态内容，考虑用程序生成，或者用单元素做法（带 SVG 文字的 CSS 遮罩，或者 Canvas）。
- **GSAP**：直接给填充层元素的 `clipPath` 做动画。`gsap.to(fillLayer, { clipPath: 'inset(0%)', duration: 0.9, ease: 'power2.inOut' })`。

## 另见
- [文字渐变动画](../text-gradient-animation/) — 颜色在字母里流动，而不是填满
- [裁剪揭示文字](../text-clip-path-reveal/) — 整行用同样的方式被揭开
- [剪裁路径揭示](../../02-entrance-and-exit/clip-path-reveal/) — 一个形状揭开任何元素
