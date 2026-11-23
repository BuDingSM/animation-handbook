# 汉堡菜单切换

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/hamburger-menu-toggle/)

## 它是什么
三条线的“汉堡”图标，激活时变成叉号。同样的三条线表示两种状态：上下两条线向中心旋转，形成叉号，中间那条淡出。它表示隐藏的菜单已经打开，而不用换成另一个图标。

## 什么时候用
- 紧凑的顶栏，完整的导航列表放不下，最常见于手机或窄平板布局。
- 任何面板、抽屉或侧边菜单，你想让触发器本身反映它的打开/关闭状态。
- 想用一个控件同时传达“点一下打开”和“点一下关闭”，而不用第二个按钮的情况。

## 工作原理
线条是三条绝对定位的 span，叠在固定高度的盒子里。切换按钮上的 `aria-expanded` 会翻转样式：外侧的线条移动到垂直中心，旋转 ±45°，中间的线条把透明度降下来。因为线条只改变 `transform` 和 `opacity`，变形留在合成器上；周围按钮的边框会用颜色过渡变成橙色，菜单则用 `opacity` 和 `transform` 打开（再加上 `visibility`，这样关闭的菜单就碰不到了）。

```css
.bar{transition:transform var(--dur) var(--ease),opacity calc(var(--dur)*.6) var(--ease)}
/* Half the 36px box minus half a line, so the lines meet at any thickness */
.burger[aria-expanded="true"] .bar.top{transform:translateY(calc(18px - var(--bar-h)/2)) rotate(45deg)}
.burger[aria-expanded="true"] .bar.mid{opacity:0;transform:scaleX(.2)}
.burger[aria-expanded="true"] .bar.bot{transform:translateY(calc(var(--bar-h)/2 - 18px)) rotate(-45deg)}
```

`translateY` 的值必须等于每条外侧线条到盒子中心的距离，这样两条线才能精确相遇；`transform-origin:center` 让旋转的支点保持在那个相遇点上。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 变化要花多长时间：慢是 550ms，普通 340ms，快是 200ms；低于大约 200ms 会显得突兀，高于 500ms 会拖沓 |
| 手感 | 轻柔 | 轻柔缓入缓出；回弹让叉号稍微过头，再落定；平滑慢慢停下；均匀保持一个稳定的速度 |
| 线条粗细 | 中等 | 线条 2px、4px 或 6px 粗；线条越粗，在小尺寸下越好读 |

## 生产注意事项
在一个真正的 `<button>`（不是 `<div>`）上切换 `aria-expanded`，在“打开菜单”和“关闭菜单”之间更新 `aria-label`，这样辅助技术能播报当前状态；用 `aria-controls` 把按钮和面板连起来。把线条标记为 `aria-hidden="true"`，因为它们是装饰性的。用 `prefers-reduced-motion` 时，把过渡切成即时切换。如果 `translateY` 的偏移是写死的，改变图标大小或线条粗细会让叉号错位——按盒子高度推导它。Framer Motion 用每个线条的 `animate` 变体来表达同样的东西；纯 CSS 版本可以用一个隐藏的复选框（`:checked ~`）驱动变形，而不用 JavaScript，代价是失去 ARIA 状态。

## 另见
- [开关滑动](../toggle-switch/) — 在开和关之间滑动的开关
- [抽屉滑出](../drawer-slide/) — 这种按钮常常打开的侧边面板
- [弹窗展开](../modal-expand/) — 窗口从你按下的按钮处长出来
