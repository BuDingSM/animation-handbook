# 联系人列表飞入飞出

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/fly-in-fly-out-contact-list/)

## 它是什么

飞入飞出的列表会在行经过滚动区域边缘附近时给它们做动画。底部进入的行淡入、上滑并放大到完整尺寸；顶部离开的行反过来做；中间的行保持不动。因为它跟着滚动位置走，而不是只播放一次，往回滚时行会以同样的方式回来。

## 什么时候用

- 联系人列表、动态和收件箱视图：用户浏览时长列表需要视觉结构
- 任何靠空间提示帮助用户在内容中定位的垂直滚动列表
- 消息应用的聊天列表、通知栏、搜索结果

## 工作原理

对视口附近的每一行（由 `IntersectionObserver` 跟踪），单个 `requestAnimationFrame` 回调会算出：

```js
const pos = (row.offsetTop - stage.scrollTop) / stage.clientHeight;
// pos ≈ 0: row top aligns with viewport top
// pos ≈ 1: row top aligns with viewport bottom
```

三个区域（Z 是区域大小，默认 0.25）：

```js
if (pos < Z) {             // exit zone (top)
  const t = pos / Z;       // 0 = fully exited, 1 = entering active
  opacity = lerp(0, 1, t);
  ty      = lerp(-DIST, 0, t);
  scale   = lerp(SCALE, 1, t);
} else if (pos > 1 - Z) { // entry zone (bottom)
  const t = (1 - pos) / Z; // 0 = fully below, 1 = entering active
  opacity = lerp(0, 1, t);
  ty      = lerp(DIST, 0, t);
  scale   = lerp(SCALE, 1, t);
} // else: active zone — opacity=1, ty=0, scale=1
```

一个 `rootMargin: '50% 0px'` 的 `IntersectionObserver` 维护着当前在视口内或附近的行的集合。rAF 回调只遍历这个集合——绝不会无条件遍历全部 15 行。唯一的例外是比观察器边距更长的跳跃（回到顶部、拖动滚动条）：它还没加进来的行还带着上次留下的样式，所以每行都会被单独处理一次。第一帧就算这样一次跳跃，所以第一幅画面就有效果。

列表末尾有 70px 的内边距，和行能滑动的最远距离一样长。没有它，底部还在进入的行会落在列表下面，拉长滚动长度，落定后又还回去；而只读一次长度的脚本（比如演示的“播放”按钮）会瞄准一个并不存在的终点。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 效果 | 三种都有 | 行在区域里做哪些变化：三种都有（移动、淡入淡出和缩小）放在一起读起来像真实的物理运动；只移动或只淡入淡出感觉更平 |
| 边缘区大小 | 中 | 每个区域有多高：小是盒子的 15%，中是 25%，大是 35% |
| 行移动的距离 | 中 | 行在最边缘处滑动多远：短是 20px，中是 40px，远是 70px |
| 行缩小的程度 | 中 | 行在最边缘处的大小：一点是 97%，中等是 94%，很多是 88% |
| 显示区域 | 关 | 给两个区域上色，离开用红色，进入用绿色 |

## 生产注意事项

- **持续效果与一次性显现。** 常见模式是元素第一次进入视口时触发一次显现动画，之后不再反转。这个演示是双向的——往上滚会恢复飞入状态。这种区别对使用场景很重要：一次性显现适合阅读流程；持续效果适合用户反复上下浏览的导航型列表。
- **性能模式。** `IntersectionObserver` 把工作集过滤到只剩视口附近的行。rAF 回调接着应用 `transform + opacity`——只走合成器的属性，永远不会触发布局或绘制。两者结合，让每帧成本与可见行数成正比，而不是与列表总长度成正比。
- **真实案例。** iOS 通讯录、Telegram 的聊天列表和 Notion 的侧边栏都在用这种进入/退出时淡入淡出加位移的变体。缩放成分让它感觉像真实的物理运动，而不是平面的。
- **库的等价写法。** GSAP ScrollTrigger 用 `scrub: true` 加每行一条时间线，可以实现同样的分段控制。Framer Motion 的 `useScroll` 加每个元素的 `useTransform` 是 React 里的等价写法；它会自动处理 `IntersectionObserver` 的记录工作。
- **无障碍。** 位移加缩放的组合是已知的前庭触发因素。在 `prefers-reduced-motion: reduce` 下，完全禁用位移和缩放；只在区域边缘用细微的透明度淡入淡出（0.3 → 1），让列表在空间上保持稳定。

## 另见

- [滚动显现](../reveal-on-scroll/) — 卡片越过一条线时出现一次
- [交错显现](../stagger-reveal/) — 一组里的元素依次出现
- [封面卡片变固定顶栏](../cover-card-to-fixed-header/) — 高封面缩小成细顶栏
