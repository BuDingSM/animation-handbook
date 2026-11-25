# 滑动删除

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/swipe-to-dismiss/)

## 它是什么
滑动删除在你把列表项横向拖动时删掉它，是邮件和消息应用里滑动删除的手势。拖动时，卡片精确地跟随你的手指；松开时，它要么飞出边缘、它那行合上，要么弹回原位。拖得长或甩得快，都能删掉它。

## 什么时候用
- 邮件、聊天和通知列表，“删除”或“归档”是主要操作
- 项目是短暂的卡片信息流（可关闭的提示、已清除的提醒）
- 任何以触摸为主的列表，每行一个按钮会显得拥挤或太慢
- 对撤销友好的破坏性操作——把飞出去和一个“撤销”小吃条配对

## 工作原理
每张卡片都用**指针事件**拖动，这样一条代码路径覆盖鼠标、触摸和触控笔。`pointerdown` 捕获指针并记录起始 X；`pointermove` 把 `translateX` 设为差值，并随距离淡出透明度；`pointerup` 决定结果：

```js
card.addEventListener('pointermove', e => {
  if (!dragging) return;
  dx = e.clientX - startX;
  const now = performance.now();
  if (now > lastT){ vel = (e.clientX - lastX)/(now - lastT); lastX = e.clientX; lastT = now; }
  follow(card, reveal, dx);
});

function follow(card, reveal, x){
  card.style.transform = `translateX(${x}px)`;
  card.style.opacity = String(Math.max(0.3, 1 - Math.abs(x)/(card.offsetWidth*0.9)));
  reveal.classList.toggle('show', showReveal && Math.abs(x) > 8);
  reveal.classList.toggle('from-left', x > 0);  // the Delete label sits on the side the card uncovers
}

function end(){
  const w = card.offsetWidth;
  const pastDist = Math.abs(dx) > w * threshold;   // committed by distance
  const flick    = Math.abs(vel) > 0.6 && Math.abs(dx) > 24 && Math.sign(vel) === Math.sign(dx); // or by velocity
  (pastDist || flick) ? dismiss(row, card, dx >= 0 ? 1 : -1) : spring(card, reveal);
}
```

确认由距离**或**速度决定——快速一甩也算数，即使手指没走多远，这正是手势感觉灵敏的原因。删除时，卡片获得一个 `flung` 过渡，完全滑出屏幕，同时包裹它的 `.row` 把它的 `height` 动画到 `0`，合上空隙，列表随之收拢。卡片上的 `touch-action: pan-y` 让垂直页面滚动通过，横向拖动由 JS 认领；浏览器接管手势去滚动时会发送 `pointercancel`，卡片总会弹回来。演示还让聚焦的消息可以用 Delete 键删掉，这样拖动不是唯一的办法。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 删除距离 | 中等 | 松开删除前，卡片必须被拖多远：短是它宽度的 20%，中等 35%，长是 50%；快速一甩（每秒快于 600px，至少拖过 24px）可以从更短的拖动删掉它 |
| 显示删除标签 | 开 | 拖动时，卡片后面显示一条带“删除”字的红色条 |

## 生产注意事项
- **用 `setPointerCapture`**，这样即使手指在拖动中途离开卡片的框，`pointermove`/`pointerup` 也会继续触发；没有它，快速滑动会丢事件。
- **`touch-action`** 很关键：在可拖动元素上设置 `pan-y`（如果你也处理垂直方向，用 `none`），否则浏览器的原生滚动会和你的拖动打架，手势在手机上会卡顿。
- **合上行，而不是卡片。** 把外层包裹的高度动画到 `0`；卡片还在飞的时候动画它自己的高度，会造成回流卡顿。变换/透明度留在卡片上，布局合上留在行上。
- **演示为什么动画高度**：列表合上就是效果本身，所以行的高度归零，这是只动画变换和透明度的例外；它保持便宜，因为一次只有一个短行在变，卡片本身用变换和透明度移动，高度一到零行就被删掉。
- **减弱动态效果**：保留*功能*，去掉缓动的 flourish——在 `prefers-reduced-motion: reduce` 下，删掉飞出去的过渡，立刻合上行，什么都不滑动。
- **破坏性滑动总要提供撤销**；没有恢复路径的飞出去，会惩罚手滑的人。只有撤销窗口关闭后，才从 DOM 里删掉。
- **库的等价物**：Framer Motion 的 `drag="x"` 加 `dragConstraints`、`onDragEnd` 和一个 `AnimatePresence` 退出，可以声明式地处理这个；`react-swipeable` 暴露 `onSwipedLeft`/速度处理函数，如果你想要手势回调而不要变换 plumbing；SortableJS 覆盖这个手势的排序表亲；GSAP 的 `Draggable` 加 `throwProps`/惯性，给出基于物理的甩动和回弹。

## 另见
- [抽屉滑出](../drawer-slide/) — 另一个由手势移动的面板
- [弹窗展开](../modal-expand/) — 从按下它的按钮处长出来的窗口
- [开关滑动](../toggle-switch/) — 滑动、落定的更小控件
- [卡片组滑动](../card-deck-swipe/) — 一叠牌的顶卡向左或向右，下一张升起
