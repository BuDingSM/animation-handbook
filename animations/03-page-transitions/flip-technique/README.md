# FLIP 技巧

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/flip-technique/)

## 它是什么
FLIP 是 First、Last、Invert、Play 的缩写，给 CSS 自己做不了动画的布局变化做动画，比如给网格排序或从多列切换到列表。浏览器先把每个条目一下子移到新位置；然后每个条目被移回原来的位置，再从那里滑到新位置。看起来条目像是滑进了新布局，但实际上只有它们在屏幕上的位置做了动画。

## 什么时候用
- 列表或网格的重新排序、排序、筛选或打乱
- 切换布局（3 列 → 列表），条目同时改变位置和尺寸
- 添加或删除条目，让邻居滑过去补位
- 任何「布局变了、我想让它动起来」而 `transition` 单独处理不了的情况

## 工作原理
四个步骤对应四块代码。**First：**记录每张卡片的包围矩形。**Last：**应用新布局，让卡片跳到最终位置。**Invert：**对每张卡片，计算新旧位置之间的差值，应用一个 `transform`，在视觉上把它送回起点。**Play：**把那个 transform 过渡到零，让卡片滑到它真正所在的位置：

```js
// FIRST — record positions before the change
const first = {};
cards.forEach(c => { first[c.id] = c.getBoundingClientRect(); });

// LAST — apply the new layout (cards jump instantly)
renderCards();
grid.offsetHeight; // force reflow

// INVERT — transform each card back to its original spot
const last = newCards.map(c => c.getBoundingClientRect());
newCards.forEach((c, i) => {
  const f = first[c.id];
  const dx = f.left - last[i].left, dy = f.top - last[i].top;
  c.style.transform = `translate(${dx.toFixed(1)}px, ${dy.toFixed(1)}px)`;
  c.style.transition = 'none';
});

// PLAY — apply the inverted positions, then animate the transform to zero
grid.offsetWidth; // force reflow
newCards.forEach(c => {
  c.style.transition = `transform ${dur}ms ${ease}`;
  c.style.transform = '';
});
```

卡片在重新渲染前后靠一个稳定的 `id` 匹配，所以一张从一个网格位置移到另一个位置的卡片能被正确跟踪。在 Invert 和 Play 之间读一下 `offsetWidth`，会让浏览器在向零过渡开始前先应用倒置的（起始）位置。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 卡片滑到位需要多长时间：慢是 800ms，普通 500ms，快 300ms；300 到 600ms 读起来是刻意的 |
| 感觉 | 平滑 | 平滑减速到停；弹性会滑过每个位置一点再回落；均匀保持一个稳定的速度 |

## 生产注意事项
- **只有 `transform` 做动画。**这就是全部要点——`transform` 和 `opacity` 对合成器友好，跳过布局和绘制，所以 FLIP 能以 60fps 给几十个重排的元素做动画，而过渡 `width`/`top` 会卡顿。
- **读一次、写一次。**把所有 `getBoundingClientRect` 读操作（First）批量放在任何样式写操作（Last/Invert）之前，避免布局抖动。在交错循环里读写会强制反复同步布局。
- **尺寸变化需要缩放。**这个演示用 `translate` 给位置做动画；当卡片同时改变尺寸（列表模式）时，从旧尺寸到新尺寸加上 `scaleX`/`scaleY`，否则尺寸变化会突然跳变而不是滑动。
- **动画卡片上的 `will-change: transform`** 提示合成器提前把它们提升到自己的图层，让第一帧更平滑——这里用在 `.flip-card` 上。
- **等价的库。**Framer Motion 的 `layout` 属性为布局变化自动实现 FLIP（它的 `AnimatePresence` 处理离开的条目）。GSAP 的 Flip 插件是直接的、开箱即用的实现。View Transitions API 通过在前后截取快照，原生实现类似的布局变化动画，无需手动算矩形。

## 另见
- [共享元素过渡](../shared-element-transition/) — 跨页面对一个元素用同样的方法
- [变形过渡](../morph-transition/) — 形状变化而不是移动
- [弹性过渡](../elastic-transition/) — 移动的弹簧收尾
- [View Transitions API](../view-transitions-api/) — 浏览器自己就能给布局变化做动画
- [拖动排序](../../04-micro-interactions/drag-to-reorder/) — 拖动一个条目到位时，条目滑到一边让路
