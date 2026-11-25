# 区域擦除切换

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/section-wipe/)

## 它是什么
区域擦除切换用下一个全屏区域从上面滑上来，取代前一个区域，而不是把旧的滚走。每个区域都吸在屏幕顶部，位于前一个之上，所以滚动时下一个区域盖住它。被盖住的区域可以略微缩小，看起来像沉到到来的那个后面。

## 什么时候用
- 全屏故事讲述区域，应该感觉是层叠的而不是滚动的
- 落地页，其中每个区域都是一块独立的“屏”，取代上一块
- 作品集或案例研究布局，区域之间有强烈的分隔
- 任何希望盖住的动作读出深度感的地方，一块面板滑到另一块后面

## 工作原理
擦除本身不需要动画代码：每个 `.section` 都是 `position: sticky; top: 0; height: 100cqh`（一个舞台高，因为舞台是尺寸容器），它们的 `z-index` 依次递增（1、2、3、4）。滚动时，每个区域吸在顶部，直到下一个区域——在层叠中更高——盖住它。唯一用脚本实现的效果是后退缩放，根据下一个区域已经盖住多少视口来计算：

```js
sections.forEach((s, i) => {
  if (i === sections.length - 1) return;              // top section never recedes
  const nextR = sections[i + 1].getBoundingClientRect();
  const coverage = sr.height - (nextR.top - sr.top);  // px of this section covered
  const t = clamp(coverage / (sr.height * wipeRange), 0, 1);
  contents[i].style.transform = `scale(${lerp(1, scaleFloor, t).toFixed(4)})`;
});
```

这里 `sr` 是舞台的内框，即它的顶边和在边框内的高度。因为 `t` 是盖住程度的连续比率，缩放是按滚动 scrub 的——在擦除中途停住，区域就停在中间的缩放比例，没有吸附。缩小让被盖住的区域感觉像滑到到来的那个*后面*，而不只是被藏起来。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 缩小 | 一点 | 被盖住的区域缩到多小：无保持原大小，一点是 96%，很多是 88%；越小沉得越深 |
| 缩小速度 | 渐进 | 缩小占用擦除的多少：渐进铺满整个擦除，中等占前 60%，快占前 30% |

## 生产注意事项
- **擦除是免费的**：吸顶定位加递增的 `z-index` 就能用零 JavaScript 做出整个盖住的动作。只为缩放这类点缀用脚本，绝不要为擦除本身用脚本。
- **缩放内容，而不是区域**：变换应用在吸顶区域内的 `.sec-content` 上，而不是吸顶元素本身——在某些引擎中，变换吸顶元素会破坏它的吸顶效果。
- **`will-change: transform`**：设在被缩放的内容上，让浏览器把它提升为独立图层；演示在减弱动态效果下会去掉它，避免占用不必要的图层。
- **保护最顶上的区域**：最后一个区域的 `z-index` 最高，永远不会被盖住，所以在缩放循环里跳过它——它应该保持缩放 1。
- **减弱动态效果**：设置了 `prefers-reduced-motion` 时，缩放循环整个跳过；区域仍然擦除（那是纯布局），但没有后退。
- **库的等价实现**：GSAP ScrollTrigger 在缩放补间上用 `scrub: true` 可以实现同样的后退，还能加上仅靠 CSS 在这里做不出的缓动；Framer Motion 把 `useScroll` 的进度映射到 `scale` 运动值，用于 React 版本。

## 另见
- [吸顶区域](../sticky-section/) — 一个区域固定不动，内容随之变化
- [层叠卡片](../stacking-cards/) — 卡片滚动时叠成一摞
- [封面卡片变固定顶栏](../cover-card-to-fixed-header/) — 大封面滚动时收成细顶栏
- [滚动吸附](../snap-scrolling/) — 滚动一次停在一个完整区域上
