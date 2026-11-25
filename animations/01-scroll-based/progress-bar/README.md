# 阅读进度条

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/progress-bar/)

## 它是什么

阅读进度指示器显示读者读到文章的哪个位置。它随滚动填满，最后一行进入视野时正好填满。演示展示了同一个想法的三种样式：顶部的一条横条、角落里的一个圆环和侧边的一条细轨。

## 什么时候用

- 长篇文章、文档页面和教程，读者受益于知道自己读到哪里了
- 有明确起点和终点的单页演示或报告
- 任何完成信号能改善阅读体验的可滚动内容区
- 进度传达“快做完了”的新手引导流程

## 工作原理

公式是恒定的，不管哪种指示器样式在用：

```js
scroller.addEventListener('scroll', () => {
  if (!ticking) { requestAnimationFrame(update); ticking = true; }
}, { passive: true });

function update() {
  const p = scroller.scrollTop / (scroller.scrollHeight - scroller.clientHeight);
  topBar.style.transform   = `scaleX(${p})`;          // top bar
  sideFill.style.transform = `scaleY(${p})`;          // side rail
  circFg.style.strokeDashoffset = CIRC - (p * CIRC);  // circle (CIRC = 2πr)
  ticking = false;
}
```

离结尾一个像素内时，值被设为正好 1，这样分数滚动位置永远不会让指示器差一丝没填满。

**指示器定位。** 三个指示器都是 `.stage`（滚动容器周围的 `position: relative` 盒子）的 `position: absolute` 子元素，而不是滚动容器本身的子元素：

```html
<div class="stage">                      <!-- position: relative; overflow: hidden -->
  <div id="top-bar"></div>               <!-- absolute, top:0, left:0, right:0 -->
  <div id="side-rail">...</div>          <!-- absolute, top:0, right:0, bottom:0 -->
  <svg id="circ">...</svg>               <!-- absolute, top:8px, right:8px -->
  <div class="scroller" id="scroller">   <!-- overflow-y: auto -->
    <div class="article">...</div>
  </div>
</div>
```

`overflow: hidden` 会把三个指示器都裁剪到圆角 `.stage` 的角落里。指示器还用了 `pointer-events: none`，所以在它们上方转动滚轮或触摸，仍然能滚动到下面的文章。

**圆形指示器**用 SVG 的 `stroke-dashoffset`：

```js
const CIRC = 2 * Math.PI * 16; // circumference = 100.53 for r=16
circFg.style.strokeDashoffset = CIRC - (p * CIRC);
// at p=0: offset = CIRC → ring fully hidden
// at p=1: offset = 0   → ring fully shown
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 指示器 | 顶部条 | 顶部条最显眼；角落里的圆环占的地方最少，还显示百分比；侧轨最安静 |
| 粗细 | 中 | 细是 2px，中是 3px，粗是 6px；细的几乎看不见，粗的很难错过 |
| 颜色 | 蓝色 | 填充的颜色 |

## 生产注意事项

- **从内部视口读滚动位置，而不是 `window.scrollY`。** 如果文章在自定义滚动容器里（就像这个演示），`window.scrollY` 永远是 0——页面本身从不滚动。公式必须用实际滚动元素的 `scroller.scrollTop` 和 `scroller.scrollHeight / scroller.clientHeight`。
- **指示器绝不能放在滚动容器里面。** `overflow-y: scroll` 容器里的 `position: absolute` 元素会跟着内容一起滚动——用户阅读时它们就消失了。把指示器放在 `position: relative` 的包装器里，它是滚动容器的兄弟（或父级），而不是子元素。
- **顶部条用 `transform: scaleX()`，而不是 `width`。** 给 `width` 做动画会让每帧都重新计算布局。`scaleX` 由合成器提升，在主线程之外运行。设置 `transform-origin: left`，让它从前缘长出来。
- **rAF 脏标志是强制的。** 滚动事件触发得比显示刷新更快（手机上 60–120 Hz）。没有 `ticking` 标志，DOM 写入会在每帧堆积多次，在低端设备上造成视觉撕裂。
- **Framer Motion 等价写法：** `const { scrollYProgress } = useScroll({ container: ref }); scaleX: scrollYProgress` 应用到一个 motion div。GSAP：`gsap.to(bar, { scaleX: () => scrollProgress, ease: "none", scrollTrigger: { scrub: true } })`。

## 另见

- [滚动 scrub 动画](../scrub-animation/) — 同样的滚动进度移动一架飞机，而不是填满一条横条
- [吸顶区域](../sticky-section/) — 滚动进度让一个区域一步步经过它的状态
- [滚动侦测导航](../scrollspy-nav/) — 菜单显示你正在读哪个章节
