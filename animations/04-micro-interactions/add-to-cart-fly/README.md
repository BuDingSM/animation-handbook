# 加入购物车飞入

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/add-to-cart-fly/)

## 它是什么
加入购物车飞入是指，访客按下「加入购物车」时，把商品图片的一个小复制品送进购物车图标。复制品在飞行中缩小，落下时购物车轻轻抖一下，购物车上的数字加一，这样访客能看到商品去了哪里。

## 什么时候用
- 网店里的商品列表和网格，购物车图标离商品很远
- 餐饮和生鲜订购，人们会连续加很多件商品
- 心愿单和稍后保存按钮
- 任何只有角落数字变化作为反馈的添加按钮

## 工作原理
按下时，商品图片被克隆，盖在自己的位置上，放在舞台的一个图层里。复制品套在两个嵌套的包裹层里，这样每个方向都有自己的时间，三个部分都用 `element.animate()`（Web Animations API）做动画，只动画 `transform` 和 `opacity`：

```js
fly.animate([{ transform: 'none' }, { transform: `translateX(${dx}px)` }],
  { duration: dur, fill: 'forwards', easing: 'linear' });                   // 横向，匀速
lift.animate([
  { transform: 'none', easing: 'cubic-bezier(.333,.667,.667,1)' },          // 上升，减速
  { offset: at, transform: `translateY(${top}px)`, easing: 'cubic-bezier(.333,0,.667,.333)' }, // 下落，加速
  { transform: `translateY(${dy}px)` }
], { duration: dur, fill: 'forwards' });
copy.animate([
  { transform: 'none', easing: 'ease-out' }, { offset: .3, transform: `scale(${mid})` },
  { offset: .85, transform: `scale(${end})`, opacity: 1 }, { transform: `scale(${end * .6})`, opacity: 0 }
], { duration: dur, fill: 'forwards' });
```

两个竖直方向的缓动是抛物线的两半：`cubic-bezier(.333,.667,.667,1)` 正好是 t(2 − t)，`cubic-bezier(.333,0,.667,.333)` 正好是 t²，所以配合横向的匀速移动，复制品走的就是一颗被扔出去的球的路径。弧顶在购物车上方稍高一点，到达弧顶的时刻 `at` 来自这样一抛所需要的起始速度：`v = 2·top − 2·√(top·(top − dy))`，`at = −v / (2·(dy − v))`。直线路径则给两个包裹层都用同一个 `ease-in-out`，这样复制品就走直线。

竖直方向动画的 `finished` promise 兑现后，复制品被移除，数量加一，购物车和它的徽标短暂地抖一下。因为落下要等动画本身完成，慢动作（把舞台里每个动画的 `playbackRate` 设为三分之一）会让飞行和落下一起变慢。重置会取消还在空中的复制品，这样它就不会迟到了。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 飞行速度 | 普通 | 复制品到达购物车要花多长时间：慢是 1.1s，普通 0.7s，快 0.45s |
| 路径 | 弧线 | 弧线横向匀速移动，同时升起再落进购物车，像抛过去一样，弧顶比横向距离的二十分之一（10 到 28px）高出一点；直线走一条直线，缓入缓出 |
| 购物车抖动 | 开 | 购物车短暂地倾斜变大，徽标在复制品落下时弹一下；关时，只有数字变化 |

## 生产注意事项
- **在按下时测量**：按下按钮时用 `getBoundingClientRect()` 读取图片和购物车的位置，因为页面可能已经滚动过；在整个页面上，给复制品用 `position: fixed`，这样飞行中滚动页面也不会移动它。
- **立刻更新购物车**：马上把商品加进购物车数据，只延迟可见的数字，这样访客中途离开也不会丢失，并通过实时区域向读屏软件播报这个变化。
- **多次按下**：每次按下都有自己的复制品；清空购物车或切换视图时，取消还在空中的复制品（`animation.cancel()`）。
- **保持简短**：飞行时间远超过一秒，会让快速购物的访客等反馈；在一个人们要加几十件商品的页面上，简单的数字变化更体贴。
- **减弱动态效果**：在 `prefers-reduced-motion: reduce` 下，什么都不飞，购物车也不抖；新数字淡入。按钮的按压也不做缓动：真实按压会立刻切换到按下的样子（没有过渡），「演示一下」的按压只短暂保持那个样子。
- **库**：GSAP 的 MotionPathPlugin 可以让元素沿曲线飞行（`motionPath: { path: [...], curviness: 1.25 }`），Framer Motion 可以给 `x` 和 `y` 用不同的 `ease` 值做出同样的弧线，CSS 的 `offset-path` 属性是沿曲线走的另一种做法。

## 另见
- [点赞爱心迸发](../heart-burst/) — 另一个庆祝按压的按钮
- [通知徽标脉冲](../badge-pulse/) — 吸引眼球看向数字的徽标
- [按钮按压缩放](../button-press-scale/) — 「加入购物车」按钮做的小按压
- [共享元素过渡](../../03-page-transitions/shared-element-transition/) — 一张图片从一个地方移到另一个地方
