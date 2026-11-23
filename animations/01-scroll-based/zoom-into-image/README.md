# 图片缩放进入

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/zoom-into-image/)

## 它是什么
图片缩放进入从图片上一个小窗口开始，滚动时打开它填满屏幕，好像你飞进了图片里。图片本身从不长大：只是揭开更多部分，所以保持清晰。窗口的圆角在打开时变方，最后淡入一段说明。

## 什么时候用
- 主视觉到内容的过渡，预览卡片应该打开成一张沉浸式图片
- 编辑或画廊开头，“进入”一张特色照片
- 区域分隔，用一张图片作为页面两部分之间的传送门
- 任何希望有*进入*图片而不是经过图片的感觉的时刻

## 工作原理
图片放在高区域内的 `position: sticky` 容器里，给出一段滚动预算。这段预算上的进度是一个 0–1 的值；内边距百分比和圆角半径都从起始值 `lerp` 到零，所以裁剪扩大的同时，圆角框也变方：

```js
const pinStart = zoomSec.offsetTop, budget = zoomSec.offsetHeight - stageH;
const p = clamp((st - pinStart) / budget, 0, 1);
const inset  = lerp(startPct, 0, p);      // 30% → 0%  (crop opens)
const radius = lerp(brStart, 0, p);       // 16px → 0  (corners square off)

zoomImg.style.clipPath = `inset(${inset.toFixed(2)}% round ${radius.toFixed(1)}px)`;

const cap = p > 0.88 ? ((p - 0.88) / 0.12).toFixed(3) : '0';
caption.style.opacity = cap;
frameBorder.style.opacity = (1 - cap).toFixed(3);   // the starting-frame outline gives way to the caption
```

裁剪而不是缩放是关键选择：`clip-path` 以全分辨率揭开更多*已有*图片，所以不像 `transform: scale()` 缩放那样变模糊或像素化。说明只在滚动最后 12% 内淡入，此时图片基本已满幅；淡的起始框轮廓在同一段内淡出，所以它从不与说明交叉。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 起始窗口 | 中等 | 起初显示图片的多少：小是盒子五分之一宽的窗口，中等五分之二，大五分之三 |
| 边角 | 圆角 | 窗口起初的边角有多圆：方形、16px 或 32px；窗口打开时它们变直 |
| 显示起始框 | 开 | 淡轮廓留在窗口开始的地方，让你看到它打开了多远；它随说明淡入而淡出 |

## 生产注意事项
- **裁剪，不要缩放**：`clip-path: inset()` 揭开真实像素，让图片每一步都保持清晰。`scale()` 缩放会放大固定渲染并变软。整幅图片都在、你只是在揭开它时，用裁剪。
- **`will-change: clip-path`**：设在图片上，让浏览器为动画裁剪做准备。在现代引擎上，`clip-path` 动画对合成器友好，但仍然受益于这个提示。
- **吸顶提供固定**：高区域滚动时，图片通过 `position: sticky` 保持不动；只有裁剪在变化，这让效果很便宜。
- **`round` 关键字**：`inset(x% round Ypx)` 把裁剪和圆角合并在一个属性里，所以两者随同一个进度值一起动画。
- **减弱动态效果**：在 `prefers-reduced-motion` 下，CSS 强制 `clip-path: inset(0%)`，立即显示说明并隐藏起始框轮廓——读者直接得到最终的完整图片，没有飞入。
- **库的等价实现**：GSAP ScrollTrigger 用 `scrub` 补间 `clipPath` 是直接对应的实现，还能加缓动；Framer Motion 在 React 中用滚动进度的 `useTransform` 动画 `clipPath` 样式。

## 另见
- [吸顶区域](../sticky-section/) — 这个效果建立在其上的固定
- [滚动 scrub 动画](../scrub-animation/) — 滚动位置驱动运动，来回都可以
- [封面卡片变固定顶栏](../cover-card-to-fixed-header/) — 大封面随滚动改变形状
- [视差景深](../parallax-depth-of-field/) — 图层随滚动移动和模糊，营造深度
- [无限缩放](../../06-3d-advanced/infinite-zoom/) — 视图自己不断潜入画中画
