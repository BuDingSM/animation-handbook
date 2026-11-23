# 封面卡片变固定顶栏

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/cover-card-to-fixed-header/)

## 它是什么

封面卡片变固定顶栏让页面以一个高封面开场，放着标题、副标题、日期和作者，滚动时再把它收成一条细顶栏。从封面高度、标题大小，到淡出的背景和小作者徽章，每个变化都跟着一个随滚动从 0 到 1 的数字走，所以中间的每一刻看起来都有安排。

## 什么时候用

- 文章页：丰富的首屏应该让位给常驻导航顶栏，又不让读者觉得突兀
- 产品详情页（例如 Apple、Stripe）：首屏图片折叠成吸顶工具栏
- 需要保留身份信息（标题、作者）又不让整个封面占滚动空间的编辑类网站和博客

## 工作原理

一个滚动监听器读取 `stage.scrollTop` 并算出：

```js
const p = Math.min(1, Math.max(0, scrollTop / RANGE));
const e = easeOutCubic(p);
```

然后在单个 `requestAnimationFrame` 回调里，根据 `e` 算出封面的可见高度和其他七个值：

```js
const h = lerp(FULL, 56, e), d = FULL - h;   // FULL: 85% of the box's height
cover.style.transform = `translateY(${-d}px)`;   // the cover keeps its full height and slides up
coverTitle.style.transform = `scale(${lerp(1, 13 / 26, e)})`;
coverBg.style.opacity      = lerp(1, 0.12, e);
coverBg.style.filter       = `blur(${lerp(0, 6, e)}px)`;
coverMeta.style.opacity    = 1 - clamp(e * 3, 0, 1);  // also coverSub
coverAuthor.style.opacity  = 1 - clamp(e * 2, 0, 1);
headerChip.style.opacity   = clamp((e - 0.5) * 2, 0, 1);
coverRule.style.opacity    = e;
```

标题用 `transform: scale()`、以 `transform-origin: left top` 缩小，而不是改写 `font-size`——原因见下面的生产注意事项。什么都不重新排版：封面在布局里保持完整高度，向上滑动收起的部分；文章跟着它一起滑；徽章和背景移回可见部分；封面比文字矮之后，文字从顶栏顶部开始。

滚动本身并不直接设定 `e`：它设定一个目标值，循环把实际绘制的值移向目标，所以滚轮的一格会滑过去而不是跳过去。每一帧走完剩余距离的一个份额，这个份额取决于这一帧花了多久，所以顶栏在 30、60 或 120Hz 屏幕上都在相同时间内落定：

```js
const dt = last ? Math.min(now - last, 50) : FRAME;  // FRAME = 1000 / 60; the first frame after a restart counts as 1/60 s
last = now;
current += (target - current) * (1 - Math.pow(1 - EASE, dt / FRAME));  // EASE 0.16 is the share covered in 1/60 s
```

回到顶部和从结尾处播放会立刻设定缓动后的值。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 缩小距离 | 中 | 滚动多远后封面完全变小：短是 200px，中是 320px，长是 440px |
| 中途跳变 | 关 | 在距离一半处直接切换高封面和细顶栏，而不是随滚动缩小；减弱动态效果时总是这样 |

## 生产注意事项

- **擦除（scrub）胜过二值切换。** 在滚动阈值处硬切换 class 会产生别扭的跳变——如果用户正好停在阈值处，顶栏会闪烁。擦除到一个 0→1 的值，意味着每个滚动位置都有明确、有意的外观。Medium、Substack 和 NYT 应用都在用这种模式的变体。
- **移动封面，绝不要给 `height` 做动画。** 写 `height` 会让页面每帧都重新排版。让封面保持完整高度，用 `translateY()` 滑动它和下面的内容，再用 `scaleY()` 从顶部边缘挤压背景——就像演示里做的那样。
- **关闭滚动锚定。** 这是每个做折叠顶栏的人都会踩的坑。封面缩小时，视口上方的内容失去了高度，浏览器会“好心”地调整 `scrollTop`，让正在看的内容保持在原地。这会降低滚动进度，进度又让封面长回来，封面又移动滚动——折叠进行到一半就卡住，滚动感觉像在跟你较劲。在滚动容器上加 `overflow-anchor: none` 就能解决。任何改变流内内容尺寸的滚动驱动动画都需要这一条。这个演示用 transform 移动封面，所以滚动永远不会改变它的高度；这条规则留着，是为了在窗口大小变化后重新测量盒子时做保障。
- **缓动最终值，而不是原始进度。** 鼠标滚轮一次约 100px；320px 的范围意味着每格就是三分之一的动画，而 easeOutCubic 在零附近的斜率是 3，会把第一格变成 68% 的折叠。缓动*输出*可以限制顶栏每帧能走多远，不管曲线多陡；缓动输入则会让曲线把步进重新放大。
- **绝不要每帧写 `font-size` 或 `box-shadow`。** `font-size` 会重塑文本行（这里测得它是单次最昂贵的写入）；`box-shadow` 会重绘。用带 `transform-origin` 的 `transform: scale()`，再淡入一条静态细线元素的 `opacity` 来代替。
- **真实案例。** Medium 的文章顶栏、Apple 的产品详情页和 Stripe 的博客都实现了这种变形的变体。区分优劣的永远是中间滚动状态看起来是有意的还是意外的。
- **库的等价写法。** GSAP ScrollTrigger 用 `scrub: true` 加一条设置每个属性的时间线，就能处理这种模式，每个属性还能单独缓动。Framer Motion 的 `useScroll` + `useTransform` 是 React 里的等价写法。两者都能让你沿着滚动时间线定义关键帧，而不用手写插值算术。
- **无障碍。** 在 `prefers-reduced-motion: reduce` 下，完全跳过擦除变形——滚动超过 `RANGE / 2` 时立刻切换到折叠后的顶栏。相反或复杂的动效会让前庭功能障碍用户感到不适；这种模式尤其会触发这个问题，因为属性在滚动过程中持续变化。

## 另见

- [固定动画](../pin-animation/) — 一部分固定不动，页面从它旁边滚过
- [层叠卡片](../stacking-cards/) — 卡片滚动时叠成一摞
- [滚动 scrub 动画](../scrub-animation/) — 滚动让动画正放倒放
- [滚动隐藏顶栏](../hide-on-scroll-header/) — 顶栏往下滚时滑走，往上滚时回来
