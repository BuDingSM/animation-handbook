# 滚动 scrub 动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scrub-animation/)

## 它是什么

Scrub 动画没有播放按钮，也没有以秒计的长度：你滚动了多远，决定了动画正好显示哪一刻，就像拖视频的播放头。往下滚就前进；往回滚就后退。演示预先画出整条飞行路线，飞机飞过时把它填满，下面有一条横条和帧数，所以你总能知道自己在哪里。

## 什么时候用

- 读者控制节奏、而不是看一段视频的产品展示
- 值得中途停下仔细看的技术讲解——机械结构、架构图
- 每次滚动推进一点故事的苹果风格功能导览
- 任何必须完全可逆、在每个中间状态都清晰可读的动画

## 工作原理

一切都来自一个数字。进度是滚动位置除以可滚动距离，取上下限：

```js
const p = clamp(stage.scrollTop / maxScroll, 0, 1);
const e = ease(p);                 // the curve is optional; the mapping is the point
```

位置和朝向都从路径本身读取，所以画出的轨迹和移动的物体永远不会对当前帧在哪里有分歧：

```js
const d  = pathLength * e;
const pt = rail.getPointAtLength(d);
// heading from two samples either side, so the object banks into its turns
const a  = rail.getPointAtLength(d + 1.5), b = rail.getPointAtLength(d - 1.5);
const angle = Math.atan2(a.y - b.y, a.x - b.x) * 180 / Math.PI;

flyer.setAttribute('transform', `translate(${pt.x} ${pt.y}) rotate(${angle})`);
trail.style.strokeDashoffset = pathLength - d;   // elapsed portion of the route
```

轨迹用标准的线条描绘技巧：`stroke-dasharray` 设为完整路径长度，`stroke-dashoffset` 从它开始倒数，进度推进时露出笔画。

这里没有任何东西基于时间：演示下面的播放按钮只是让盒子匀速滚动，画每一帧的是同一份代码。滚动是位置的一个输入，而不是播放的触发器。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 感觉 | 均匀 | 滚动如何变成进度：均匀一对一跟随滚动；轻柔进出都缓和；平滑向结尾放慢；慢启动开始慢、然后加速 |

## 生产注意事项

- **Scrub 是位置，不是播放。**导致 bug 的心智模型是把滚动当作*播放*触发器，然后再想办法反转它。每帧从进度计算状态，反转就是免费的——没有方向要跟踪，也没有动画对象要倒带。
- **永远不要按状态开关更新。**区域“不活跃”时跳过计算，会留下它恰好停在的值。改为把进度取上下限到 `0–1`，帧就能正确落定，不管读者怎么到达，包括直接跳到结尾。
- **给读者整条时间线。**抽象 scrub 场景的失败模式是观众不知道自己在哪里、还剩多少，运动读起来像噪音。预先画出整条路线再把它填满，就把动画变成了它自己的进度条。
- **布局读取不要进帧循环。**`getTotalLength()`、`scrollHeight` 和 `clientHeight` 只测量一次，尺寸变化时刷新。每帧读它们会在每次滚动事件时强制同步重排。
- **等效的库。**GSAP ScrollTrigger 用 `scrub: true` 是生产标准，`scrub: 0.5` 加一个平滑延迟，让滚轮刻度的滚动感觉不那么一格一格。Framer Motion 的 `useScroll` + `useTransform` 是 React 里的等效做法；两者都是对这个进度到值的映射的封装。
- **无障碍。**页面到达时滚动一次盒子，在 `prefers-reduced-motion: reduce` 下完全不滚动；之后只有访客滚动时它才动，所以这是直接操作，而不是强加的运动。在减弱动态效果下，演示去掉装饰性的光晕、保留映射，因为完全去掉它会让控件变得死板而不是更平静。

## 另见

- [滚动触发动画](../scroll-trigger/) — 动画在设定的滚动点开始、跟随并固定
- [固定动画](../pin-animation/) — 一部分固定不动，页面从它旁边滚过
- [阅读进度条](../progress-bar/) — 同样的滚动进度，显示为一条填满的横条
