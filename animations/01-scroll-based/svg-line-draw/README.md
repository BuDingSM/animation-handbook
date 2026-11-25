# SVG 线条滚动描绘

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/svg-line-draw/)

## 它是什么

滚动描绘的线是一条长长的蜿蜒路线，你往下滚时它自己画出来，像在地图上描绘一段旅程。沿途带标签的站点在线到达时弹出，往上滚又会擦掉线。路线的一份淡虚线副本显示它要去哪里。

## 什么时候用

- 旅程或路线故事讲述——旅行日志、探险地图、货运跟踪
- 时间线，每个里程碑应该在读者到达时出现
- 流程讲解（“你的订单如何送到你手里”），一条连接线给步骤一个空间顺序
- 需要一条视觉脊柱把各区域串起来的长篇落地页

## 工作原理

整个效果是两个描边属性和一个数字。初始化时测量一次路径，把 `stroke-dasharray` 设为它的全长——描边现在是一条恰好与路径等长的单条虚线，`stroke-dashoffset` 把它滑入视野：

```js
const LEN = path.getTotalLength();          // measure ONCE, at init and on resize
path.style.strokeDasharray = LEN;

// per scroll frame (coalesced with requestAnimationFrame):
const p     = ease(clamp(stage.scrollTop / maxScroll, 0, 1));   // Feel: p for Even, 1 - (1 - p)^3 for Smooth
const drawn = Math.min(LEN, LEN * p / COMPLETE_AT);          // COMPLETE_AT below 1 finishes before the scroll does
path.style.strokeDashoffset = LEN - drawn;                   // reveal the traveled portion
```

`COMPLETE_AT` 默认是 1，所以线随滚动完成，笔尖在用均匀绘制时留在盒子内（平滑在滚动早期画得靠前）。更小的值让线更早完成：剩下的滚动显示完成的路线，但笔尖跑到盒子前面去了，因为路线有好几个盒子高。

站点放在路径已知的分数位置上，所以它们的位置和触发点只根据这些分数计算一次——绝不在每个滚动事件里计算：

```js
// at init: position each waypoint and store its length threshold and where its dot sits in the scrolling content
wp.len = LEN * wp.fraction;
const pt = path.getPointAtLength(wp.len);
wp.group.setAttribute('transform', `translate(${pt.x} ${pt.y}) scale(${k})`);
wp.y = svgTop + pt.y / k;

// per frame: two cheap comparisons, class toggled only when the state changes
const on = drawn >= wp.len && wp.y - 16 < scrollTop + boxHeight;
if (on !== wp.on) { wp.on = on; wp.group.classList.toggle('on', on); }
```

每个站点还按 `k` 缩放，`k` 是 600 除以绘图在屏幕上的宽度，所以它的圆点和标签在每块屏幕上保持同样大小，而路线本身缩放。

站点还会等到它进入视野，也就是它的顶边在盒子底边之上。路线有好几个盒子高，所以线可以画到盒子前面很远，在底边下面弹出的站点会在没人能看到的地方弹出；这样它的弹出在它滚进来时播放。静止图片两种情况都一样，因为盒子下面的站点看不见。

弹出本身是 CSS——`.on` 把站点从 `scale(.6)` 和 `opacity: 0` 过渡到完整尺寸，往上滚过站点会去掉这个类，让它取消弹出。跳跃之后——回到顶部、播放从顶部重新开始，或一帧内移动超过半个盒子——过渡会为那一次更新关掉，所以没有站点会在引向它的线消失后才淡出。进度是钳制的，从不设门：直接跳到底部会画出整条路线，回到顶部会擦掉它，中间没有任何会卡住的状态。路径的一份淡虚线副本放在绘制描边的下面，所以读者总能在线到达之前看到路线要去哪里。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 感觉 | 均匀 | 均匀让线尖与你的滚动保持齐平；平滑一开始画得快、临近结尾放慢 |
| 显示站点 | 开 | 五个带标签的站点在线到达时弹出，往上滚时再次藏起 |
| 线完成于 | 在结尾 | 滚动进行到多远时线完成：提前在 70%，接近结尾在 85%，在结尾在 100%；提前完成会在你到达底部之前显示整条路线 |

## 生产注意事项

- **只测量一次。**`getTotalLength()` 会强制做几何计算；在滚动处理器里调用它（或读 `scrollHeight`）会导致布局抖动。在初始化时缓存长度和最大滚动量，在防抖后的 resize 时刷新，每帧的工作只做纯算术加一次样式写入。
- **绝不要给每帧都写的属性加 CSS 过渡。**`stroke-dashoffset` 在每个动画帧都设置；再加 `transition: stroke-dashoffset ...` 会让浏览器向一个每 16ms 就移动的目标做动画——线会落后于滚动并拖影。过渡属于状态变化（站点弹出），不属于 scrub 的值。
- **`pathLength="1"` 完全跳过测量。**在路径上设置属性 `pathLength="1"`，就可以用 `stroke-dasharray: 1` 并直接写 `stroke-dashoffset = 1 - p`——完全不用调用 `getTotalLength()`。这个演示要测量，因为站点反正需要真实坐标。
- **`vector-effect: non-scaling-stroke`** 在 SVG 响应式缩放时保持描边宽度不变；没有它，窄手机上渲染的 3px 描边可能细成一根发丝。
- **标签可读性。**在 `<text>` 上用 `paint-order: stroke` 加背景色的描边，会在字形下面画一圈轮廓，所以标签在强调线从后面经过的地方仍然可读——比滤镜或衬底矩形更便宜、更清晰。
- **库的等价实现。**GSAP 的 DrawSVGPlugin 配 ScrollTrigger（`scrub: true`）是生产标准；Framer Motion 的 `useScroll` + `pathLength` 运动值是 React 的等价实现。两者包装的正是这个 dashoffset 映射。

## 另见

- [滚动 scrub 动画](../scrub-animation/) — 一条短的固定路径随你滚动移动；这里一条长路径在内容里
- [SVG 路径动画](../../06-3d-advanced/svg-path-animation/) — 绘图随时间自己描绘，而不是随滚动
- [滚动叙事](../scrollytelling/) — 文字旁边的图片随故事滚动而变化
