# 圆形揭示

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/circle-reveal/)

## 它是什么
圆形揭示通过一个圆圈展示下一页，圆圈从一个点开始，不断变大，直到新页面填满整个视图。圆点位于访客点击或轻触的地方，所以新页面看起来像是从他们的手指下扩散开来。返回时可以反向运行：旧页面缩回那个点，露出更早的页面。

## 什么时候用
- 应用风格的导航，新视图应该从打开它的按钮里长出来
- 主题切换，深色或浅色版本的页面从开关处扩散开来
- 圆形菜单按钮和浮动操作按钮，打开成一个完整的面板
- 任何点击位置带有含义的变化：这张卡片、这个按钮、地图上的这个点

## 工作原理
两个页面在同一个框里。移动的页面被放在上面，并裁剪成一个圆：打开时，新页面从点击处以零半径的圆开始变大；关闭时（打开「返回时收回」后返回），旧页面留在上面，它的圆缩小到零。结束尺寸是到框最远角落的距离，在变化开始时量出：

```js
const w=area.clientWidth,h=area.clientHeight;
const x=origin==='center'?w/2:origin==='corner'?w:pt.x,y=origin==='center'?h/2:origin==='corner'?h:pt.y;
const far=Math.ceil(Math.max(Math.hypot(x,y),Math.hypot(w-x,y),Math.hypot(x,h-y),Math.hypot(w-x,h-y)));
const closing=closeTog.checked&&next<prev,top=closing?oldEl:newEl,at=`px at ${x.toFixed(1)}px ${y.toFixed(1)}px)`;
top.style.zIndex='2';top.style.transition='none';top.style.clipPath='circle('+(closing?far:0)+at;
top.offsetHeight; // commit the starting circle before the change is switched on
top.style.transition=`clip-path ${d}ms cubic-bezier(.4,0,.2,1)`;
top.style.clipPath='circle('+(closing?0:far)+at;
```

点击位置来自指针事件（`clientX` 和 `clientY`，减去框自己的角）。键盘点击没有位置（它的 `detail` 是 0），所以圆圈从被按按钮的中间开始。一个页面名位于页面框上方，所以从那里开始的圆圈在顶边稍上方开始，向下扩散。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 起始位置 | 点击处 | 点击处跟随指针，键盘按压时用按钮中间；中央和角落（右下角）总是在同一个地方开始 |
| 速度 | 普通 | 圆圈盖住页面需要多长时间：慢是 1200ms，普通 750ms，快 450ms |
| 返回时收回 | 开 | 去往更早的页面时，旧页面缩回那个点，而不是让更早的页面从那里长出来 |

## 生产注意事项
- **到达最远的角落。**一个和框一样宽的圆会让角落一直到最后才被盖住。从起点到最远角落的距离是能盖住一切的最小半径，所以变化在最后一个角落到达时正好结束。
- **强制回流。**在起始圆和过渡之间读一下 `offsetHeight`，会让浏览器先确认圆点；没有它，两次写入会合并，页面一下子出现。
- **在点击的那一刻量位置。**框可能在两次点击之间移动（滚动、调整尺寸），所以在点击处理程序里读它的位置，永远不要在加载时读一次。
- **`clip-path` 便宜但不是免费的。**圆圈变大时，浏览器会重绘被裁剪的页面。让被裁剪的页面保持轻量，避免在它上面用重的滤镜或阴影。
- **减弱动态效果**改为在 300ms 内淡入新页面；什么都不长大。
- **等价的库。**View Transitions API 原生实现这一点：用 `document.startViewTransition()` 开始切换，然后在 `::view-transition-new(root)` 上给 `clip-path` 做动画，从点击处的圆到完整半径（常见的深色模式开关）。GSAP 和 Framer Motion 直接给 `clipPath` 字符串做动画。

## 另见
- [传送门 / 隧道缩放](../portal-zoom/) — 圆圈从一个圆形按钮打开
- [剪裁路径揭示](../../02-entrance-and-exit/clip-path-reveal/) — 一个不断变大的形状揭示单个元素
- [点击涟漪](../../04-micro-interactions/click-ripple/) — 一个圆环从按压处扩散开来
- [交叉淡入淡出过渡](../crossfade/) — 它回退到的普通淡入淡出
