# 滚动隐藏顶栏

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/hide-on-scroll-header/)

## 它是什么

滚动隐藏顶栏是阅读时让路的顶栏。往下滚时它向上滑出视野，内容得到整个屏幕；往上滚时它回来，所以菜单总是一小步之遥。靠近页面顶部时它始终显示。

## 什么时候用

- 移动端网站：固定不动的顶栏会占去小屏幕的一大块
- 新闻网站、博客和长文章：读者大多往下滚，只是偶尔想要菜单
- 顶栏里放着搜索和购物车的商店和目录，访客想快速找回它们，而不用一路滚回顶部
- 任何在人们阅读时盖住太多内容的吸顶顶栏

## 工作原理

顶栏放在页面上方（演示里它盖在滚动盒子上；在真实页面里它是 fixed 或 sticky），内容获得顶栏高度的顶部内边距，这样没有任何东西从它下面开始。滚动监听器自己不做任何工作：它只要求在下一动画帧做一次更新。更新把滚动位置和上一次起作用的位置比较。往下移动藏起顶栏，往上移动显示它，靠近顶部时始终显示。几像素的移动会被忽略，而且只有移动算数时才更新上一次的位置，所以慢速滚动也能累积。

```js
function update() {
  ticking = false;
  const y = Math.min(Math.max(scroller.scrollTop, 0), maxY); // the rubber-band bounce at either end does not count
  if (y <= nearTop) { setAway(false); lastY = y; return; }   // near the top: always shown
  const dy = y - lastY;
  if (Math.abs(dy) < JITTER) return;                         // a few pixels of jitter: ignored
  if (dy > 0) setAway(true);                                 // down: away
  else if (comeBack === 'any') setAway(false);               // up: back (with "Only near the top", only the top does it)
  lastY = y;
}
scroller.addEventListener('scroll', () => {
  if (!ticking) { ticking = true; requestAnimationFrame(update); }
}, { passive: true });
```

`setAway()` 只切换一个 class。样式表用 transform 移动顶栏，所以下面的页面永远不会重排；缩小的版本让顶栏留一条细条在视野里，并把内容缩放到能放进去：

```css
.topbar { position: absolute; top: 0; left: 0; right: 0; transition: transform var(--dur) var(--ease); }
.topbar.away { transform: translateY(calc(-100% - 2px)); }                 /* its full height, border included, and 2px to spare */
.shrink .topbar.away { transform: translateY(-18px); }                     /* a slim strip stays */
.shrink .topbar.away .brand { transform: translateY(9px) scale(.82); }    /* centered in the strip */

@media (prefers-reduced-motion: reduce) {
  .topbar { transition: opacity var(--dur) linear; }
  .topbar.away { transform: none; opacity: 0; }                            /* it fades instead of sliding */
}
```

“演示一下”会滚动盒子本身，往下滚一个半盒子再回来，每帧的位置都根据滑动开始以来的时间算出，所以在 120 Hz 屏幕上和在 60 Hz 屏幕上花的时间一样。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 滑动花的时间：慢是 700ms，普通 300ms，快 150ms |
| 回来方式 | 任何往上滚动时 | 任何往上滚动时在第一次往上移动时就把顶栏带回来；只在顶部附近则一直藏着，直到页面回到离顶部两个顶栏高度之内 |
| 缩小而不是隐藏 | 关 | 留一条细顶栏在视野里，内容更小，而不是把整个顶栏滑走 |

## 生产注意事项

- **吸顶或固定**：在真实页面上用 `position: fixed`（页面加顶部内边距）或 `position: sticky`，并把 `scroll-padding-top` 设为顶栏高度，这样顶栏显示时，指向标题的链接和获得焦点的表单字段就不会被顶栏盖住。
- **保留阈值**：触控板和触摸屏会报告很多微小的移动和小幅反转。没有几像素的容差，顶栏会闪烁进出。
- **钳制位置**：iOS 的橡皮筋滚动在顶部报告负位置，在底部报告超出末尾的位置。从末尾弹回来的那一下否则会被读成往上滚动，把顶栏弹回来。
- **键盘用户**：焦点进入顶栏时显示它（`:focus-within`），否则用 Tab 键走到菜单的访客会落在看不见的链接上。
- **只用 transform**：绝不要给 `top`、`margin` 或 `height` 做动画。用 transform 移动顶栏可以让它留在合成器上，缩小的版本缩放内容而不是改变字号。
- **减弱动态**：把顶栏淡出淡入，或者留在原地，而不是滑动它。
- **不要藏起关键操作**：顶栏里放着页面主按钮（结账、撰写、保存）时，应该留着它。
- **库**：Headroom.js 正是做这个的，有阈值和离顶部偏移量的选项；在 React 或 Vue 里，一个小的滚动方向 hook 也能做到。较新版本的 Chromium 增加了滚动状态容器查询，可以告诉你盒子上一次往哪个方向滚，正是为这个设计的，但纯 CSS 在每个浏览器里还做不到，所以方向检查留在这里的 JavaScript 里。

## 另见

- [封面卡片变固定顶栏](../cover-card-to-fixed-header/) — 高封面滚动时收成细顶栏
- [滚动侦测导航](../scrollspy-nav/) — 菜单高亮你正在读的章节
- [阅读进度条](../progress-bar/) — 往下读时进度条填满
- [平滑（惯性）滚动](../smooth-scroll/) — 滚动滑到停下，而不是一下跳到位
