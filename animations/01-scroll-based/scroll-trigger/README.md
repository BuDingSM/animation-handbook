# 滚动触发动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-trigger/)

## 它是什么

滚动触发在你滚动时，在选定的点开始、跟随或固定动画。页面的每个区域要么还在前面、要么正在屏幕上、要么已经过去，在这些状态之间移动会启动或反转它的动画。演示展示了四种常见用法：内容到达时淡入、把动画绑定到滚动上、卡片固定时变化、点亮步骤。

## 什么时候用

- 任何进入视口时应该正好播放一次的元素
- 需要滚动协调时机的产品展示、功能列表和区域过渡
- 需要靠合理的静止状态、在没有 JavaScript 时仍可访问的营销页面
- 任何需要在滚动边界处精确回调、而不只是“它可见吗？”的场景

## 工作原理

一切都来自一个数字：滚动容器当前的位置。每个区域归结为三种状态，四个回调就是它们之间的转换。

```js
// zone tops measured against the stage, cached — NOT offsetTop, see production notes
const state = i => {
  const vt = stage.scrollTop;
  if (vt + viewport <= top[i]) return 'idle';   // not reached yet
  if (vt >= top[i] + height[i]) return 'past';  // scrolled beyond
  return 'active';
};
```

状态变化就是一个回调。是哪一个取决于状态移动的方向（演示通过区域 1 的文字展示它们，进入时淡入，离开时藏起）：

```js
if (s !== prev[i]) {
  const ev = s === 'active' ? (prev[i] === 'idle' ? 'onEnter' : 'onEnterBack')
                            : (s === 'past'       ? 'onLeave' : 'onLeaveBack');
  prev[i] = s;   // run the zone's enter or leave animation for ev
}
```

Scrub 用的是同一个滚动位置，归一化到 `0–1` 并取上下限，然后直接应用到变换上。因为它是取上下限而不是按激活状态开关，所以在窗口之前是 0，之后是 1——于是跳过的区域最终还是会落在完成状态。窗口属于播放动画的部件（方形、那排圆点），而不是它周围的区域：这里一个区域将近一盒高、内容居中，所以按区域顶部定的窗口会在部件能被看见之前就跑完：

```js
// follow window: 0 as the piece's center passes 80% down the box, 1 at 25%
// (mid: the middle of its layout box, measured against the stage and cached like the zone tops)
const p = clamp((stage.scrollTop - (mid - 0.8 * viewport)) / (0.55 * viewport), 0, 1);
swatch.style.transform = `rotate(${p * 180}deg) scale(${lerp(1, 1.3, p)})`;
swatch.style.filter    = `hue-rotate(${p * 120}deg)`;
```

固定区域是例外。它在*被固定*的距离上 scrub，而不是在它进入的窗口上，所以它的进度按区域与视口之间的溢出量来度量：

```js
// pin span: 0 as the zone top meets the container top, 1 as its bottom meets the bottom
const p = clamp((stage.scrollTop - top[i]) / (height[i] - viewport), 0, 1);
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 区域高度 | 盒子的 90%（区域 3：两倍盒子） | 每个区域需要多少滚动 |
| 跟随窗口 | 从盒子 80% 处到 25% 处 | 区域 2 的方形和区域 4 的那排圆点在各自的中心升到这两条线之间时播放，所以每个都在完整视野里播放 |
| 固定跨度 | 区域 3 的高度减去一盒 | 卡片固定不动多久；它的三个阶段平分这段时间 |
| 尾部空间 | 一盒高 | 最后一个区域后面的空间，让它能完成并离开 |

## 生产注意事项

- **GSAP ScrollTrigger** 是生产标准，通过声明式 API 处理所有这些：滚动方向、固定、scrub、吸附和生命周期回调。演示是手工实现的，为了展示底层机制，没有 GSAP 的优化。
- **`IntersectionObserver` 与滚动事件。**IO 是生命周期回调（进入/离开）的正确工具；滚动位置算术适合 scrub。这个演示把两者都从滚动位置推导出来，让它们保持同步——用 IO 的话，回调和 scrub 可能差一帧。
- **把 scrub 锚定在区域*进入*的地方，而不是它到达顶部的地方。**按 `scrollTop − zoneTop` 度量进度，会让它一直是零，直到区域顶部一路爬到视口顶部——到那时读者已经看了一整屏它一动不动的样子，动画只在它离开时才播放。这读起来像坏掉或严重迟到的触发器。一个常见的 GSAP 设置，`start: "top 80%"` / `end: "top 20%"`，在区域出现后不久开始，在它落定时结束；在这里它把每个触发器大约提前了 0.8 个视口。也要把它锚定到播放动画的部件上：一个将近一盒高、内容居中的区域，会把部件放在区域顶部下方半盒处，所以演示在方形和那排圆点上用 `start: "center 80%"` / `end: "center 25%"`，动画在能被看见的地方播放。
- **`offsetTop` 不在滚动容器的坐标空间里。**它是从最近的*定位*祖先开始度量的，对于普通的 `overflow: scroll` 面板来说通常是 `body`——所以它包含容器上方页面框架的每一个像素，而 `scrollTop` 在容器内部从零开始。直接比较它们会让这里的每个触发器都晚 109px。改为按容器自己的盒子度量区域。
- **触发回调之前先初始化状态。**把第一帧的状态与 `null` 的初始值比较，会凭空制造出从未发生过的事件：区域 1 会在任何滚动之前触发 `onEnterBack`，区域 2–4 会触发 `onLeaveBack`。在第一次遍历时记录初始状态，之后只发出转换。
- **每帧评估每个触发器，不只是激活的那些。**只在区域激活时更新它，会留下它恰好停在的值——跳过一个区域，它的 scrub 根本不会运行，滚回顶部时固定的卡片还停留在“第 3 步，共 3 步”。取上下限的进度在区域之前是 0，之后是 1，所以两端都能自己正确落定。
- **在最后一个触发器后面留一段尾部空间。**下面什么都没有的最后一个区域永远滚不过顶部，所以既不能完成 scrub，也不能触发 `onLeave`——没有它，区域 4 的圆点永远没法全部亮起来。一个视口的尾随空间就够了。
- **固定区域需要有地方可固定。**元素的包含块比滚动容器短时，`position: sticky` 什么都不做——没有溢出量可以让它固定着走。区域 3 特意做成两个视口高，让卡片有一个视口的行程可以固定；没有它，卡片会像其他卡片一样直接滚过去，三个步骤在它上屏之前就全触发了。如果它和滚动器之间的某个祖先有 `overflow: hidden`，sticky 也会静默失效。
- **Scrub 与 `will-change`。**对于每滚动一帧就更新的元素，在第一帧之前声明 `will-change: transform`，避免提升时的卡顿。
- **吸附到点**需要检测滚动静止。最后一次滚动事件后的防抖定时器（100–200ms）是可靠的模式；大多数浏览器环境里没有原生的“滚动结束”事件。
- **无障碍。**`onEnter` 动画应该遵循 `prefers-reduced-motion`。在减弱动态效果模式下，页面加载时直接把元素跳到最终状态。

## 另见

- [滚动 scrub 动画](../scrub-animation/) — 滚动让动画正放倒放
- [固定动画](../pin-animation/) — 一部分固定不动，文字随之变化
- [滚动显现](../reveal-on-scroll/) — 卡片越过一条线时出现
