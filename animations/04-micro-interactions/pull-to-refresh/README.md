# 下拉刷新

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/pull-to-refresh/)

## 它是什么
下拉刷新在你从列表顶部把它往下拉时重新加载列表。拉的时候，列表跟着手指走，但越往下拉阻力越大，上面一个旋转指示淡入、转动。拉过一段固定距离再松开，列表会停在下面，旋转指示继续转动，新内容加载进来；然后它滑回去，新内容在最上面。

## 什么时候用
- 信息流、收件箱和时间线，“获取最新”是频繁、预期的操作
- 以触摸为主的屏幕，可见的重新加载按钮会让页眉显得拥挤
- 内容在两次访问之间会从服务器变化的列表（通知、订单、聊天）
- 任何内容已经可以从顶部滚动的地方，这样手势有开始的空间

## 工作原理
列表放在一个滚动器里，和上面旋转指示的区域一起，由一个 `translateY` 移动。在 `pointerdown` 时记录这个指针的起始 Y；`pointermove` 时，只有手指往下移动、滚动器已经在顶部（`scrollTop <= 0`），手势才会被*认领*，这样其他地方的正常滚动不受影响。原始拖动距离经过一个橡皮筋函数：列表一开始和手指一比一跟随，拉得越远反抗得越厉害；阻力设置只决定它变硬的速度：

```js
function resist(dy){                 // 1:1 at first, stiffer as it grows; resistance sets how fast
  const max = threshold * (1 + 2.4 / resistance);
  return max * dy / (dy + max);
}

scroller.addEventListener('pointermove', e => {
  if (e.pointerId !== pid || refreshing) return;
  const dy = e.clientY - startY;
  if (!pulling){
    if (dy > 4 && scroller.scrollTop <= 0){ pulling = true; scroller.setPointerCapture(pid); }
    else return;                     // let native scroll run
  }
  pull = resist(dy);
  setY(pull);                        // translateY on list + indicator
  drawSpin();                        // opacity + rotate scale with pull/threshold
});
```

在 `pointerup` 时，如果 `pull >= threshold`，视图进入刷新状态：列表被补间到一个保持旋转指示可见的静止偏移，旋转指示从拖动驱动的旋转切换成连续的 CSS 动画；刷新时间过后，一个新项目被加到前面，所有东西弹回去。拉得短，或浏览器取消了手势，就直接补间回零。列表页眉里的“刷新”按钮和“演示一下”按钮启动的是同一次刷新；“演示一下”用同样的 `setY()` 和 `drawSpin()` 一帧一帧画出它的拉动。只有 `transform` 和 `opacity` 在动——永远不用 `top` 或 `height`——所以整个手势留在合成器上。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 下拉距离 | 中等 | 松开刷新前，列表必须被拉多远：短是 48px，中等 64px，长是 80px，按阻力之后量 |
| 阻力 | 中等 | 拉动时列表反抗得多厉害：它一开始跟着手指走，然后变硬；要拉到下拉距离，需要大约 1.4 倍（低）、1.8 倍（中等）或 2.25 倍（高）的拖动；列表永远不会超过它的 3.4、2.2 或 1.8 倍；下拉距离长、阻力高时，需要很长的拖动 |
| 旋转指示 | 圆环 | 转动的圆环、三个闪烁的圆点或四条拉伸的竖条 |
| 刷新时间 | 中等 | 新项目到来前，旋转指示转动多长时间：短是 700ms，中等 1200ms，长是 2000ms |

## 生产注意事项
- **只在顶部认领手势。** 在劫持拖动之前检查 `scrollTop === 0`（或 `<= 0`），否则列表中间的向下滚动会被吞掉，列表会感觉卡住。
- **`touch-action` 和被动监听器。** 在滚动器上设置 `touch-action: pan-y`，这样垂直滚动仍能工作；如果你在拉动里 `preventDefault`，移动监听器就不能是 `{ passive: true }`。在边界处没有什么可滚动的，所以变换读起来很干净，不会和原生滚动打架。
- **`overscroll-behavior: contain`** 会阻止浏览器自己的过度滚动辉光/回弹，防止拉动连到页面或父级滚动器上。
- **原生浏览器下拉刷新冲突。** 移动浏览器在文档上自带下拉刷新。把你的手势放在嵌套的滚动容器里（不是页面根部），生产中在 `html`/`body` 上用 `overscroll-behavior-y: contain`，这样浏览器的重新加载不会在你的上面触发。
- **无障碍替代。** 拖动手势键盘和辅助技术够不着，所以总要把同样的操作暴露成一个真正的控件——演示里列表页眉的“刷新”按钮走的是完全相同的代码路径，一个实时区域会播报状态。遵循 `prefers-reduced-motion`，不用弹性补间，直接弹回。
- **库的等价物**：Framer Motion 可以用拖动上的 `useMotionValue` + `useTransform` 和 `onDragEnd` 阈值检查来驱动同样的拉动；PullToRefresh.js 为普通网站包装了这个模式；iOS 的 `UIRefreshControl` 和 Android 的 `SwipeRefreshLayout` 是原生的平台等价物。

## 另见
- [滑动删除](../swipe-to-dismiss/) — 横向拖动，删掉一个项目
- [加载旋转指示](../loading-spinner/) — 单独的旋转指示
- [抽屉滑出](../drawer-slide/) — 从边缘滑入的面板
