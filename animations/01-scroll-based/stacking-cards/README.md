# 层叠卡片

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/stacking-cards/)

## 它是什么

层叠卡片把每张卡片固定在滚动区域顶部，下一张卡片从它上面滑过，卡片叠成一摞。下面每张卡片都留出一条可见，每张被盖住的卡片略微缩小，所以一摞看起来有深度。叠放本身不需要脚本：浏览器的吸顶定位就能做到。

## 什么时候用

- 作品集或案例研究页面，每个项目都值得拥有自己的全屏时刻
- 新手引导序列，每一步取代上一步而不是跟在它后面
- 故事讲述布局，一条信息必须完全讲完，下一条才能出现
- 任何希望滚动感觉像翻页而不是移动镜头的场合

## 工作原理

叠放本身完全不需要 JavaScript。吸顶加在 `.card` 包装器上——而不是内部内容上——按序号设置的 `padding-top` 把每张卡片的内容往下推，让上面的卡片作为一条露出的边保持可见。

```css
.card {
  position: sticky;
  top: 0;
  padding-top: calc(var(--index0) * var(--peek)); /* the peek tab */
  z-index: var(--index);                          /* later cards on top */
}
```

深度提示来自 CSS 滚动驱动动画。`#cards` 声明一个视图时间线，每张卡片在该时间线自己的片段内做动画：第 *i* 张（共 *N* 张）卡片在退出阶段的 `[i/N, (i+1)/N]` 内缩放，所以卡片一张接一张压缩，而不是同时压缩。

```css
#cards { view-timeline-name: --cards-timeline; }

.card__content {
  animation: card-scale linear both;
  animation-timeline: --cards-timeline;
  animation-range: exit-crossing calc(var(--index0) / var(--numcards) * 100%)
                   exit-crossing calc(var(--index)  / var(--numcards) * 100%);
  transform-origin: 50% 0%;   /* collapse downward, keeping the peek tab flush */
}
@keyframes card-scale {
  to { transform: scale(calc(1 + var(--scale-step) - var(--scale-step) * var(--reverse-index))); }
}
```

反向序号是让深度读起来正确的关键：最顶上的卡片（反向序号 1）落在缩放 1.0，埋在它下面的每张卡片再压缩一步。

对于不支持 `animation-timeline` 的浏览器，一个 `requestAnimationFrame` 处理器重现同样的曲线。进度从一摞顶边到达容器顶部时的 0，到它的底边到达时的 1，每张卡片取同样的 `[i/N, (i+1)/N]` 片段：

```js
const p = clamp((stage.scrollTop - cardsTop) / cardsH, 0, 1);
const t = clamp(p * N - i, 0, 1);
content.style.transform = `scale(${lerp(1, 1 + step - step * (N - i), t)})`;
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 露出的边 | 中等 | 每张卡片在下一张上面留出的一条：无、小（8px）、中等（14px）或大（20px） |
| 卡片缩小多少 | 中等 | 每张卡片上面每多一张卡片，它就小多少：一点是 5%，中等 10%，很多 12% |
| 卡片数量 | 5 | 三、五或七张卡片；越多一摞越高、滚动越长 |

## 生产注意事项

- **CSS 干活；JS 添彩。**叠放用零 JavaScript 就是正确的——卡片仅靠 `position: sticky` 和 `z-index` 就按正确顺序出现。滚动驱动时间线提供深度提示（缩放），JavaScript 路径只为不支持 `animation-timeline` 的浏览器而存在。
- **`transform: scale()` 只走合成器。**它既不触发布局也不触发绘制，所以即使每张卡片同时做动画，深度提示的成本也很低。
- **留白必须在吸顶父元素内部。**吸顶元素一旦自己的底部到达包含块的底部就会脱开。最后一张卡片的底部与一摞的末端齐平，所以它被固定的距离恰好是零像素——到头时，完成的一摞直接从顶部滑走，留下一个空舞台。把尾部空间放在容器*后面*没有用，容器上的 `padding-bottom` 也没有用：吸顶约束是包含块的**内容**框，不包括内边距。它必须是卡片后面的一个真实子元素。它的高度正好是你能继续滚动、同时把完成的一摞留在屏幕上的距离。
- **给卡片定尺寸时要算上它自己的偏移。**每张卡片是内容加上它的 `padding-top` 露出部分，最深的那张卡片承载着最高的叠加。把内容定为固定的 `stage − 80px`，意味着一旦 `(N−1) × peek` 超过这点余量，最后一张卡片就会溢出容器——7 张卡片时它被裁剪了。改成减去露出的总和（`stage − 12px − (N−1) × peek`），任何数量或露出值下每张卡片都留在舞台内。
- **绝不要从吸顶元素读 `offsetTop`。**这是这个模式里的陷阱。`position: sticky` 会改变元素的*使用*位置，所以 `offsetTop` 报告的是卡片当前吸住的位置，而不是它在流中的位置——这个值随滚动移动，滚动中途的测量会得到倒着跑的偏移。改成累加 `offsetHeight`（吸顶不影响它），并从非吸顶祖先那里取容器的原点。
- **注意坐标空间。**`offsetTop` 从最近的定位祖先（通常是 `body`）测量，而 `scrollTop` 从滚动容器的内容原点测量。混用它们会悄悄地把所有东西偏移容器上方页面框架的量。如果父元素和子元素共享一个 `offsetParent`，把它们的 `offsetTop` 相加会重复计算。
- **让回退保持诚实。**一个只是“看起来差不多”的 JS 回退比没有更糟，因为没人会再检查它。在几个滚动位置把它的输出与原生时间线对比——在这里，两者在每个点上都一致到舍入误差以内。
- **GSAP ScrollTrigger** 在每张卡片上用 `pin: true` 加一个 `scrub` 时间线，可以驱动同样的缩放提示，并且对每张卡片的缓动有更多控制。Framer Motion 的 `useScroll` + `useTransform` 在 React 中用每个元素相对于视口的进度实现同样的效果。
- **无障碍。**缩放变化很微妙，不太可能引发前庭不适。在 `prefers-reduced-motion: reduce` 下，完全禁用深度提示——仅靠 z 顺序，叠放仍然清晰可读。两条路径都要关：只杀掉 CSS 动画，对还在写内联变换的 JavaScript 回退毫无作用；而且任何已经设置过的内联变换都必须清除，否则它会冻结在 CSS 时间线之上。

## 另见

- [固定动画](../pin-animation/) — 一部分固定不动，页面从它旁边滚过
- [滚动吸附](../snap-scrolling/) — 盒子一次停在一个区域上
- [封面卡片变固定顶栏](../cover-card-to-fixed-header/) — 高封面收成细顶栏
