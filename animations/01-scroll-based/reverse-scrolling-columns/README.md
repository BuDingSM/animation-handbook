# 反向滚动列

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/reverse-scrolling-columns/)

## 它是什么

反向滚动列把一列正常滚动的列放在两列反向移动的列中间。反向运动让布局显得有深度、有活力，这是视差用到的同一个线索，只是更强了。每列放两份卡片并循环，所以列永远不会跑完。

## 什么时候用

- 大胆的视觉对比定下编辑基调的主视觉布局
- 需要动效但不想用重型素材的作品集或代理机构展示
- 页面滚动保持原生、由一个封闭舞台驱动效果的滚动驱动功能
- 任何有意用反向运动制造视觉张力的地方

## 工作原理

每列包含两组相同的卡片，上下叠放。一个 `scroll` 事件监听器读取 `stage.scrollTop`，传给一个 `requestAnimationFrame` 回调，给每列应用 `translate3d`：

```js
// Center column — travels up, the direction a normal scroll moves content
const cOff = -(scrollTop % singleSetH);
centerInner.style.transform = `translate3d(0, ${cOff}px, 0)`;

// Side columns — negative mult reverses direction; magnitude sets speed
// (moving down, margin-top pre-offsets them by -singleSetH so there is content above; moving up, it is 0)
const sideOff = (scrollTop * Math.abs(mult)) % singleSetH;
const sideY   = mult >= 0 ? -sideOff : sideOff; // negative = up, positive = down
sideInner.style.transform = `translate3d(0, ${sideY}px, 0)`;
```

`singleSetH` 是一组卡片的高度（12 张卡片加间距），在卡片生成后从 DOM 测量。在 `document.fonts.ready` 时再测一次——第一次测量后换入的网络字体会改变卡片高度，让循环周期短了几个像素，每次循环都会跳一下。取模把偏移保持在 `[0, singleSetH)` 范围内，第二组与第一组的起点正好对齐时，就能无缝循环。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 两侧列方向 | 反向 | 反向让两侧列逆着滚动流动；同向让三列一起移动，效果就消失了 |
| 两侧列速度 | 相同 | 两侧列相对于中间列的速度：更慢是一半，相同是相等，更快是一倍半；再快运动就让人不舒服了 |

## 生产注意事项

- **无限循环需要取模计算。**没有循环，列走完一遍就没内容了。两组的结构（一组可见，一组等待）保证两端始终有内容。
- **高速有前庭风险。**反向运动是已知的前庭触发因素。生产中把 `|mult|` 保持在 0.3 到 1.0 之间；避免超过 1.5 的值。始终遵循 `prefers-reduced-motion`，把 `mult = 0`（两侧列冻结）——反向运动在 WCAG 2.1 SC 2.3.3（交互动画，AAA 级）中被列为前庭危害。
- **等效的库。**GSAP ScrollTrigger 可以用 `scrub: true` 和每列自定义回调驱动列变换。Locomotive Scroll 提供每元素的 `data-scroll-speed` 属性，接受负值来做反向运动。Lenis 加一个滚动订阅者也能做到。
- **性能。**`translate3d` 把每列提升到自己的合成层。配合在首次绘制前于 CSS 中声明的 `will-change: transform` 使用。永远不要给 `top`、`margin-top` 或 `height` 做动画——它们会触发布局。

## 另见

- [视差滚动](../parallax-scrolling/) — 图层随滚动以不同速度移动
- [视差景深](../parallax-depth-of-field/) — 图层随焦点移动而移动和模糊
