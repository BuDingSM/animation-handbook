# 视差滚动

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/parallax-scrolling/)

## 它是什么

视差滚动让场景的远图层比近图层移动得少，滚动时扁平的图片看起来有了深度。它模仿了你从火车车窗看到的景象：近处的东西飞速掠过，远处的山丘几乎不动。每个图层有自己的速度，速度之间差多远，决定了场景感觉有多深。

## 什么时候用

- 落地页的主视觉区域，场景定下氛围
- 带环境深度的故事滚动和编辑专题
- 产品展示，场景上下文强化主题
- 受益于氛围沉浸感的作品集“关于”栏目

## 工作原理

每个图层有一个速度倍数 `s`，在 0 到 1.5 之间，由“深度”设置决定。滚动时，单个 `requestAnimationFrame` 回调把 `scrollTop` 归一化为 0–1 的进度值，再按图层的速度缩放移动距离（盒子高度的四分之一）：

```js
const progress = scrollTop / (scrollHeight - clientHeight); // 0 → 1
const offset   = progress * MAX_PARALLAX * speed;           // MAX_PARALLAX = a quarter of the box's height
layer.el.style.transform = `translate3d(0, ${offset}px, 0)`;
```

先归一化让移动在任何滚动长度下都一样，两端还免费得到钳制。

速度低的图层（天空：10%）几乎不动——看起来很远。速度高的图层（草地：100%）移动最多——看起来很近。速度之间的比例决定了幻觉有多可信。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 深度 | 普通 | 天空、山脊、树林和草地的速度：普通是 10%、30%、60% 和 100%；浅是 50%、65%、80% 和 100%；深是 0%、25%、70% 和 150%；平让四个一起移动。100% 时，一个图层在整个滚动中走过盒子高度的四分之一 |

## 生产注意事项

- **GPU 加速：** `translate3d`（而不是 `translateY`）把元素提升到合成器层。配合第一帧之前声明的 `will-change: transform`，避免初次滚动时图层提升造成的卡顿。
- **rAF 节流：** 滚动事件触发得比显示刷新更快（某些设备上高达 1000Hz，而帧率是 60–120Hz）。脏标志模式（`ticking` 布尔值）保证每渲染一帧最多排一个 `requestAnimationFrame`，防止重复工作。
- **库的替代方案：** GSAP ScrollTrigger 提供 `scrub` 缓动和时间线集成。Locomotive Scroll 和 Lenis 增加了带视差钩子的动量/惯性滚动。Rellax 是简单场景的轻量零依赖选择。
- **无障碍：** 遵循 `prefers-reduced-motion: reduce`。当用户要求减弱动态时，让每个图层保持零偏移，显示静态场景，并在减弱动态媒体查询里应用 `will-change: auto`，避免不必要的图层提升。

## 另见

- [视差景深](../parallax-depth-of-field/) — 同样的图层，加上移动的焦点让它们模糊
- [反向滚动列](../reverse-scrolling-columns/) — 列随滚动朝相反方向移动
- [波浪层](../../07-ambient-background/wave-layers/) — 同样的近快远慢规则，改成循环而不是滚动
