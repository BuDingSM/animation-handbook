# 分段控制器

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/segmented-control/)

## 它是什么

分段控制器是一排选项，其中恰好一个被选中，由它后面的高亮标记。选中另一个选项时，高亮滑过去，拉伸到它的宽度，同时标签变色，变化读起来像一个东西在移动。它是天、周、月这类视图选择器背后的开关。

## 什么时候用

- 在固定的一小组视图之间切换（天 / 周 / 月）。
- 按一个独占的筛选条件过滤列表，所有选项都放得进一行。
- 有 2–5 个短选项、想让它们同时都可见时，代替下拉框。
- 任何单选按钮组能用、但你想让选项横向排开、选中移动清晰的地方。

选项多、标签长或会换到多行时不要用——下拉框或标签列表扩展性更好。

## 工作原理

指示器是一个绝对定位的元素。它永远不动画 `left` 或 `width`；JavaScript 量出目标段的几何形状，完全用 `transform` 驱动运动，合成器可以动画它而不用做布局工作。`translateX` 把胶囊移到段的偏移处；`scaleX` 把一个固定的参考宽度拉伸到匹配不同宽度的段。标签的颜色过渡用同样的时长，所以交叉淡入淡出在胶囊到达时落下。

```js
function move(i, animate) {
  const target = opts[i].getBoundingClientRect();
  const base   = opts[0].getBoundingClientRect();    // reference width
  const x  = target.left - base.left + opts[0].offsetLeft;   // offset inside the track
  const sx = target.width / base.width;              // scaleX to fit this segment
  if (!animate) ind.style.transition = 'none';       // no slide on first paint
  ind.style.width = base.width + 'px';
  ind.style.setProperty('--x', x + 'px');
  ind.style.setProperty('--sx', sx);
  if (!animate) { void ind.offsetWidth; ind.style.transition = ''; }  // flush the jump, then allow sliding again
}
```

```css
.seg-ind {
  transform-origin: left center;
  transform: translateX(var(--x)) scaleX(var(--sx));
  transition: transform var(--dur) var(--ease);
}
```

因为 `transform-origin` 是左边缘，`scaleX` 从段的左边界向右把胶囊拉长，宽度变化时保持对齐。选择在量取选项之前被标记（`aria-checked`），因为选中的标签设得稍重一点，会稍宽一点。

一条跑过头一点的曲线（回弹）给高亮一种动量感；平滑读起来更克制。标签的颜色变化用和滑动同样的时长，所以两者永远不会脱节。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 高亮滑过去要花多长时间：慢是 550ms，普通 340ms，快是 200ms |
| 手感 | 回弹 | 回弹稍微跑过选项，再落定；平滑慢慢停下；均匀保持一个稳定的速度 |
| 高亮 | 胶囊 | 选中选项后面一个填满的胶囊，或它下面一条 2px 的下划线 |
| 选项数量 | 四个 | 两到五个选项；更多或标签太长，放不进一行 |

## 生产注意事项

- **语义和键盘。** 把组暴露为 `role="radiogroup"`，子项为 `role="radio"`（如果每个选项换一个面板，用 `tablist`）。用游走的 `tabindex`——只有选中的选项能被 Tab 停住——并处理方向键、Home 和 End 来移动选择。这个演示正是这样做的。
- **重新计算几何。** 宽度取决于渲染出的文字，所以在 web 字体换进来之前，胶囊可能放错位置。在 `resize` 时和 `document.fonts.ready` 里面重新运行量取步骤，每次重建段时也是。
- **RTL。** 从右到左的布局里 `offsetLeft` 仍从左边量，但阅读顺序翻转；验证方向键的方向和胶囊原点，用 `dir="rtl"` 测试，而不是想当然。
- **减弱动态效果。** 在 `prefers-reduced-motion: reduce` 下，过渡被去掉，选中会立刻跳过去——状态变化不用移动也依然清晰。
- **库。** 框架把这个包装成组件（Radix / React Aria 的 `ToggleGroup`、Material 的分段按钮、SwiftUI 的 `Picker(.segmented)`），GSAP 或 Framer Motion 可以用弹簧驱动 `transform`；底层的先量后移机制是一样的。

## 另见

- [开关滑动](../toggle-switch/) — 在两种状态之间滑动的开关
- [悬停状态动画](../hover-state/) — 指针在上面时有反应的项目
- [手风琴展开/收起](../accordion/) — 原地打开和关闭的区块
