# 平滑（惯性）滚动

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/smooth-scroll/)

## 它是什么

平滑滚动让滚动滑起来。它不把内容恰好移动滚轮每一转或每一次滑动的距离，而是把每一次当作一个要去的地方，每帧让内容靠近一点，于是先加速冲出去、然后慢慢停下。结果感觉有分量、有动量，而不是浏览器那种瞬间滚动。

## 什么时候用

- 营销和作品集网站，有意的电影感滚动节奏能强化品牌
- 滚动驱动的故事讲述，希望动画与平滑后的滚动值感觉同步
- 把平滑滚动与视差或 scrub 时间线搭配的界面，让每个与滚动关联的效果共享一个缓动后的唯一真实来源
- 你掌控整个页面体验、并能接受下面无障碍取舍的情况

不要用在密集的实用型内容（仪表盘、文档、长表单）上，用户在那里希望滚动精确跟随输入。

## 工作原理

保持两个数字：`target`（滚动要去的地方）和 `current`（内容现在的位置）。输入事件只移动目标。一个 `requestAnimationFrame` 循环每帧把 `current` 向 `target` 移动剩余距离的一部分——即线性插值，也就是 lerp——并把这个值以 `translate3d`（仅合成器变换）写到内部内容上。循环只在两者不同时运行：输入启动它，一旦 `current` 与 `target` 相差不到十分之一个像素就停止，所以静止的盒子不花任何代价。

```js
// input only nudges the target
viewport.addEventListener('wheel', e => {
  e.preventDefault();
  target = clamp(target + e.deltaY, 0, maxScroll);
  kick();                                        // starts the loop if it is idle
}, { passive: false });

// each frame eases current toward target, by a share that depends on how long the frame took
const FRAME = 1000 / 60;                         // ease is counted per 1/60 s
function loop(now) {
  raf = 0;
  const dt = last ? Math.min(now - last, 50) : FRAME;   // time since the last frame; 1/60 s for the first
  last = now;
  current += (target - current) * (1 - Math.pow(1 - ease, dt / FRAME));   // ease ~0.09
  if (Math.abs(target - current) < 0.1) current = target;
  content.style.transform = `translate3d(0, ${-current}px, 0)`;
  if (current !== target) raf = requestAnimationFrame(loop);
}
function kick() { if (!raf) { last = 0; raf = requestAnimationFrame(loop); } }
```

触摸拖动用 Pointer Events：在 `pointerdown` 时捕获当前目标，`pointermove` 按拖动距离偏移它，于是同一套 target/current 机制同时服务鼠标滚轮和手指拖动。`target` 与 `current` 之间的差距就是动量的来源：一划把 `target` 推到前面，`current` 在后面滑行追赶，直到两者汇合。

键盘移动的是同一个目标：在滑动模式下，盒子上的 `keydown` 处理器按方向键移动 40px，按 Page Up、Page Down 和空格键移动盒子的 90%，按 Home 和 End 移到顶部或底部。

演示把这一切都放在带 `overflow: hidden` 的限定盒子里——从不碰 `window` 滚动——所以可以嵌入而不接管页面。

每帧移动的份额取决于该帧花了多久。每帧固定的 lerp 系数是依赖帧率的：同样的值在 120Hz 屏幕上比 60Hz 快一倍，在 30Hz 手机上慢一半。这里 `ease` 是在 1/60 秒内走过的份额，持续了 `dt` 的一帧走过剩余距离的 `1 - Math.pow(1 - ease, dt / FRAME)`，所以剩余距离随时间指数缩小，滑动在每块屏幕上耗时相同。`kick()` 会清空 `last`，所以启动后的第一帧按 1/60 秒算，而不是按与某个旧帧相隔的时间算；`dt` 上限为 50ms，所以隐藏的标签页不会让内容跳变。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 滑动 | 中等 | 内容在六十分之一秒内走过剩余距离的多少：长是 5%，中等 9%，短 16%，所以 95% 的滚动分别在约 1 秒、0.5 秒和 0.3 秒后落定；更长的一帧走得更多，所以滑动在每块屏幕上耗时相同；长感觉沉重飘忽，短更接近普通滚动 |
| 普通滚动 | 关 | 关掉滑动，让盒子按浏览器自己的方式滚动，以便比较两者 |

## 生产注意事项

- **生产环境用真正的库。**[Lenis](https://github.com/darkroomengineering/lenis) 是当前标准；[Locomotive Scroll](https://github.com/locomotivemtl/locomotive-scroll) 比它更早。它们处理跨浏览器的滚轮归一化、触摸和触控板怪癖、锚点链接，以及手写循环会漏掉的集成钩子（例如 GSAP ScrollTrigger）。
- **不要破坏键盘滚动和锚点滚动。**原生滚动响应 Page Up/Down、空格、方向键、Home/End、Tab 聚焦和 `#anchor` 跳转。天真的接管会悄悄禁用所有这些。真正的库会重新实现它们；如果你自己写，也必须实现。
- **尊重 `prefers-reduced-motion`。**平滑滚动正是某些用户会觉得迷失方向或恶心的那种运动。当用户要求减少动态效果时，回退到原生即时滚动，不做 lerp——这个演示在这种情况下会把自己锁定为原生模式。
- **注意滚动位置的陷阱。**因为真实滚动位置是用变换伪造的，`position: sticky`、`scrollIntoView`、`IntersectionObserver` 的阈值和浏览器的滚动恢复都可能读到错误的偏移。任何依赖原生滚动几何的东西都需要改喂平滑后的值。
- **不要随意接管全局滚动。**接管 `window` 滚动会影响页内查找、滚动条的拖动行为和辅助技术。把效果限定在一个容器元素内（像这里一样）更安全；全页平滑应该是经过深思熟虑、测试过的决定。
- **SEO 和可靠性。**内容在 DOM 中，所以仍然可抓取，但任何 JS 故障都可能让页面无法滚动。确保脚本出错或从未运行时能优雅地回退到原生滚动。

## 另见

- [视差滚动](../parallax-scrolling/) — 图层以不同速度移动；与滑动搭配很好
- [滚动 scrub 动画](../scrub-animation/) — 滚动驱动的运动，滑动让它更平滑
- [横向滚动](../horizontal-scroll/) — 往下滚，一行面板横向移动
