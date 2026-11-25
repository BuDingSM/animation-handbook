# 淡入淡出

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/fade-in-out/)

## 它是什么

淡入淡出改变元素的不透明度，在看不见和完全可见之间切换：淡入出现，淡出离开。它是最简单的入场动画，也是运行成本最低的之一，因为浏览器只需要混合一个已经画好的图层。什么都不移动、也不改变形状，所以淡入淡出读起来是一次中性的出现，而不是一次移动。

## 什么时候用
- 弹窗、提示框和弹出层的出现，细微的存在变化就够了
- 两种状态之间的交叉淡入淡出（加载骨架屏 → 加载好的内容）
- 柔化任何更生硬的动作——把淡入淡出和滑动或缩放搭配起来，让元素是渐渐清晰而不是突然出现
- 路由或视图切换，想要变化但不要空间隐喻的时候

## 工作原理
卡片从 `opacity: 0` 开始，加上 `.in` 类时过渡到 `1`。时长和缓动由 CSS 变量驱动，控件可以实时改写它们：

```css
.card {
  opacity: 0;
  transition: opacity var(--dur) var(--ease);
  will-change: opacity;
}
.card.in { opacity: 1; }
```

JS 做的只是切换这个类。为了干净地重播，演示先去掉 `.in`，用 `void card.offsetWidth` 强制重排，再加回来——没有这次重排，浏览器会把去掉和加上合并成没有变化，过渡就永远不会触发：

```js
card.classList.remove('in');
void card.offsetWidth;   // flush styles so the next add restarts the transition
card.classList.add('in');
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 淡入淡出要多长时间：「慢」是 1000ms，「普通」600ms，「快」350ms；不到 150ms 的淡入淡出几乎感觉不到 |
| 感觉 | 平滑 | 「平滑」开始得快、结束得轻柔，是入场最自然的感觉；「轻柔」两头放慢；「均匀」保持一个稳定的速度 |

## 生产注意事项
- **只给 `opacity` 做动画，别的都不动。**它只走合成器，所以即使在低端手机上淡入淡出也能保持 60fps。改给 `visibility` 或 `display` 做淡入淡出则完全不会动——它们是离散属性。
- **`display: none` 不能过渡。**如果元素淡出后必须离开布局，先监听 `transitionend`（或者用 `transition-behavior: allow-discrete` 配合 `@starting-style`）再移除它，否则它会在淡入淡出完成之前就消失。
- **减弱动态：**在 `prefers-reduced-motion` 下，演示把淡入淡出缩短为 300ms 的线性交叉溶解，而不是取消它——淡入淡出本来就很轻柔，所以更快的版本仍然舒服。
- **库里的等价写法：**GSAP 的 `gsap.to(el, { opacity: 1 })` 或 `.fromTo()` 形式；Framer Motion 的 `animate={{ opacity: 1 }}`，退场淡出配合 `AnimatePresence`；Motion One 的 `animate(el, { opacity: [0, 1] })`。

## 另见
- [滑入](../slide-in/) — 淡入的同时从一边滑进来
- [缩放进入](../scale-in/) — 从小放大到完整尺寸
- [模糊渐入](../blur-in/) — 淡入时从模糊中变清晰
- [剪裁路径揭示](../clip-path-reveal/) — 一个形状揭开它，没有淡入淡出
