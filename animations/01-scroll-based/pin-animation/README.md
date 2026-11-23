# 固定动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/pin-animation/)

## 它是什么

固定动画让页面的一部分保持不动，其余部分从它旁边滚过。演示里，一台手机和它的文字在你滚动经过一个三个盒子高的区域时停在盒子中间，它的四个功能随之变化；区域结束时，它们松开手滚走。浏览器的 sticky 定位完成了固定，没有任何脚本在移动东西。

## 什么时候用

- Apple 风格的产品功能导览：一个界面元素保持可见，解说文字向前推进
- 分步新手引导：一个持久元素在多个滚动步骤中保持在视野里
- 任何视觉锚点必须稳定、周围上下文不断变化的布局
- 保持空间连续性有助于理解的教程序列

## 工作原理

固定只需要三样东西：一个高的父区域、一个吸顶子元素和匹配的高度。滚动容器是 `position: relative`，所以区域的 `offsetTop` 在滚动器内部测量，而不是以页面为准：

```css
.stage {
  position: relative;
  overflow-y: auto;
  container-type: size;   /* 100cqh is one box height */
}
.pin-section {
  height: 300cqh;         /* three times the box */
}
.pin-inner {
  position: sticky;
  top: 0;
  height: 100cqh;         /* one box tall */
}
```

JavaScript 根据在区域内的位置算出显示哪个功能：

```js
const pinStart  = pinSection.offsetTop;   // measured inside the scroll container
const pinHeight = pinSection.offsetHeight;
const viewH     = stage.clientHeight;
const p = clamp((scrollTop - pinStart) / (pinHeight - viewH), 0, 1);
const featureIndex = Math.min(3, Math.floor(p * 4));
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 固定区域 | 三个盒子高 | 手机在多出来的两个盒子高度里保持不动；更高的区域让每个功能持续更久 |
| 功能数量 | 4 | 这些功能平分固定的滚动距离 |
| 固定位置 | 盒子顶部 | 固定框架吸住的位置；手机坐在它的中间 |

## 生产注意事项

- **“多出来的高度”就是滚动预算。** `top: 0` 的固定元素在正好 `(父元素高度 − 视口高度)` 像素的滚动里保持可见。把父元素高度设为 `viewport_height + scroll_budget`。
- **GSAP 的 `pin: true`** 在 ScrollTrigger 里复制了这种行为，还增加了更平滑的功能过渡和时间线擦除。Locomotive Scroll 和 Lenis 都原生支持带惯性滚动的固定区域。
- **性能。** 固定内容并没有在动——它只是吸顶。功能切换是文字/class 的变化。除了 `position: sticky` 本身建立的东西，不需要 GPU 合成。
- **无障碍。** 内容变化时悄悄滚动的固定区域会让屏幕阅读器用户困惑。确保功能变化通过 `aria-live` 播报，或者 DOM 里的逻辑阅读顺序保持不变。

## 另见

- [滚动 scrub 动画](../scrub-animation/) — 滚动让动画正放倒放
- [滚动触发动画](../scroll-trigger/) — 动画在设定的滚动点开始、跟随并固定
- [吸顶区域](../sticky-section/) — 整个区域固定不动，内容在里面变化
