# 粘性菜单

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/gooey-menu/)

## 它是什么
粘性菜单是一个圆形按钮，按下后，较小的按钮会像液滴一样从里面渗出来，再按一次，又被拉回去。液滴是分开移动的素圆圈；一个滤镜先把它们模糊，再把模糊的边缘锐化回来，这样靠得近的圆圈会融化成一个形状，中间拉出一条液体脖颈，直到断开。

## 什么时候用
- 移动应用里的浮动操作按钮，放三四个快捷操作
- 帖子、照片和商品上的分享菜单（分享、点赞、保存）
- 活泼的品牌、游戏和作品集，带一点个性正合适的地方
- 平时应该折起来、需要时才用的工具

## 工作原理
舞台有两层。下面那层只放一种颜色的圆圈：圆形按钮下面一个大圆圈，每个小按钮各一个圆圈和一个更小的“尾巴”圆圈。只有这一层会经过一个 SVG 滤镜：

```html
<svg width="0" height="0"><filter id="goo" color-interpolation-filters="sRGB">
  <feGaussianBlur stdDeviation="9"/>
  <feColorMatrix values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 20 -9"/>
</filter></svg>
```

模糊把每个圆圈的边缘摊成柔和的坡道。颜色矩阵保留颜色，把 alpha 乘以 20 再减去 9：模糊后的 alpha 低于大约一半时，像素变成透明；高于时变成实色，柔和的边缘就又被切回硬边。靠得近的两个圆圈把柔和的边缘叠加到超过那条线，就读成一个形状，中间有一条平滑的脖颈；分开时，脖颈变细断开。模糊越大（粘稠程度），它们能够到的距离就越远。演示按菜单大小缩放模糊，手机上减少五分之一；模糊更大时，把 20 提高到模糊值的 2.2 倍，仍在 alpha 为 0.475 处切开，这样边缘保持锐利。

圆圈只用 `transform` 上的过渡来移动：

```css
.goo { filter: url(#goo); }   /* only as big as the open menu: the filter's cost grows with its area */
.goo i { border-radius: 50%; background: #ff9d5c; }
.blob { transform: translate(0, 0) scale(.5);   /* closed: waiting inside the big circle */
        transition: transform 480ms cubic-bezier(.55, -.25, .75, .25) var(--back); }
.open .blob { transform: translate(var(--x), var(--y)) scale(1);
              transition: transform 600ms cubic-bezier(.3, 1.4, .55, 1) var(--out); }
.tail { transform: translate(0, 0) scale(.4);
        transition: transform 570ms cubic-bezier(.55, -.25, .75, .25) calc(var(--back) + 72ms); }
.open .tail { transform: translate(var(--x), var(--y)) scale(.6);
              transition: transform 810ms cubic-bezier(.25, .8, .3, 1) calc(var(--out) + 60ms); }
```

打开时，每个圆圈先稍微弹出超过位置，再落定（曲线的 1.4 超过了 1），尾巴晚一点、慢一点跟上，所以有一瞬间，圆形按钮、尾巴和圆圈形成一滴拉长的液滴。关闭时，圆圈先稍微向外倾斜（曲线的 −0.25），然后越来越快地被拉进去，尾巴拖在后面。`--out` 让每个圆圈比前一个晚 60ms 启动，`--back` 关闭时反向，所以液滴一个接一个地离开、回来。大圆圈每次都会用一段短的关键帧动画胀大一次。

上面那层放真正的按钮：一个半透明的圆形按钮，上面的加号会变成叉号；还有三个带图标的小按钮，每个都和自己的圆圈用同样的变换移动。图标保持锐利，因为它们不在滤镜里，键盘也能到达它们。每个落在哪里，由舞台的大小决定：布局以 64 单位的圆形按钮、50 单位的小按钮和 20 单位的间距为单位写成，一个单位是舞台较短的一边减去 28px，再除以 274（最长的布局），保持在 0.88px 到 1.2px 之间，这样小按钮永远不会小于 44px。圆形按钮放在打开菜单的中心位置，滤镜层覆盖打开的菜单和周围 22 个单位。

```js
function setOpen(on) {
  gm.classList.toggle('open', on);
  group.inert = !on;                      // closed: out of reach of Tab and clicks at once, while the buttons still fade
  main.setAttribute('aria-expanded', String(on));
  main.setAttribute('aria-label', on ? 'Close menu' : 'Open menu');
}
main.addEventListener('click', () => setOpen(!isOpen()));
gm.addEventListener('keydown', e => {   // Escape, from any of its buttons
  if (e.key === 'Escape' && isOpen()) { setOpen(false); main.focus({ preventScroll: true }); }
});
```

关闭时，小按钮组立刻变成 `inert`（按钮只有淡出后才变成 `visibility: hidden`），所以 Tab 和点击都碰不到一个正在淡出的按钮。按下一个会执行它的事（这里一条简短消息说“已分享”“已点赞”或“已保存”，一个状态区域会读出来），关掉菜单，把焦点还给圆形按钮；按菜单旁边也会关掉它。演示一下会按下圆形按钮，再按“点赞”，而不移动焦点；按压、按键、滚轮、触摸或 Tab 进入舞台都会让它停下来。减弱动态效果下，滤镜和尾巴都关掉，什么都不移动：小按钮在原地淡入淡出。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 布局 | 弧线 | 横排把小按钮排成一行，放在圆形按钮旁边；竖排把它们叠在它上面，间距 20px；扇形展开把它们以 84px 的距离、60° 的间隔散在它上面；圆形按钮 64px 宽，小按钮 50px。所有这些都是基础缩放：整个菜单随舞台缩放 0.88 到 1.2 倍，所以最小的时候圆形按钮大约 56px 宽，最大时 77px |
| 粘稠程度 | 中等 | 边缘锐化前圆圈被模糊多少：轻微是 5px 模糊，中等 9px，强烈 13px，对应 64px 宽的圆形按钮（随菜单缩放，手机上减少五分之一）；越大，液滴连接的时间越长 |
| 速度 | 普通 | 小按钮弹出来要花多长时间：慢是 950ms，普通 600ms，快是 380ms，每个比前一个晚 90、60 或 40ms 启动 |
| 显示素圆圈 | 关 | 关掉滤镜，把每个圆圈画成圆环，展示下面移动的东西；这只是一种查看方式，所以不会加进复制的提示词里 |
| 按钮颜色 | 橙色 | 圆圈的颜色；图标保持深色 |

## 生产注意事项
- **保持滤镜小**：浏览器会在滤镜层里有东西移动的每一帧，模糊它的每一个像素。给滤镜一个打开菜单那么大的图层，永远不要整个页面，也不要把图标和文字放进去。演示把图层设为打开菜单的大小（布局变化时是两个菜单的大小），手机上模糊减少五分之一（宽至 600px 或高至 500px 的屏幕）。静止时什么都不会再被滤镜处理。
- **Safari**：HTML 元素上的 SVG 滤镜在 Chrome、Firefox 和 Safari 里都能用，但在 Safari 里可能更慢。在 iPhone 上测试；慢手机上降低模糊，或少用几个液滴。
- **图标锐利**：滤镜层里的图标会被模糊、切成一团。像这里一样，把图标放在上面的图层，或者给颜色矩阵的结果命名（`result="goo"`），再以 `<feComposite in="SourceGraphic" in2="goo" operator="atop"/>` 结束滤镜，这样原来的图会画回粘性图上面。
- **一种颜色**：`color-interpolation-filters="sRGB"` 让颜色保持精确（默认的 linearRGB 会在边缘改变颜色）。分开的圆圈上的渐变会在每个脖颈处断开，所以给整个图层一种填充。
- **切口**：滤镜处理过的圆圈结束在比真实尺寸略小一点的地方，模糊越大越明显。把画在按钮上的东西，比如光泽或边框，放在离边缘几像素以内的地方，否则会显示成一条细环。
- **无障碍**：圆形按钮是真正的 `<button>`，带 `aria-expanded`、`aria-controls` 和一个说“打开菜单”或“关闭菜单”的标签；每个小按钮都有名字；让关闭的按钮立刻变成 `inert`，而不只是淡出后，这样 Tab 不会落到访客再也看不到的按钮上；Esc 关掉菜单并还回焦点；选择的结果会被播报。
- **GSAP**：`gsap.to(blobs, { x: i => places[i][0], y: i => places[i][1], scale: 1, duration: 0.6, stagger: 0.06, ease: "back.out(1.7)" })`，按钮也一样；滤镜留在它自己的图层上。
- **Framer Motion**：给每个圆圈 `<motion.i animate={open ? { x, y, scale: 1 } : { x: 0, y: 0, scale: 0.5 }} transition={{ type: "spring", stiffness: 400, damping: 18, delay: i * 0.06 }} />`，放在 `<div style={{ filter: "url(#goo)" }}>` 里面。

## 另见
- [变形色块](../../06-3d-advanced/morphing-blob/) — 同样的粘性滤镜，用在自己漂移的圆圈上
- [汉堡菜单切换](../hamburger-menu-toggle/) — 三条线在菜单打开时变成叉号
- [搜索框展开](../expanding-search/) — 搜索图标展开成输入框，用完后收起
- [弹窗展开](../modal-expand/) — 窗口从你按下的按钮处长出来
- [按钮按压缩放](../button-press-scale/) — 按下时按钮缩小
