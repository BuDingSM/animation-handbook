# 闪光 / 漏光过渡

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/flash-transition/)

## 它是什么
闪光过渡把一次页面切换藏在一道闪光里。一层彩色层盖住页面并升到全强度，趁盖住时换页，然后这层再淡出，露出新页面。眼睛会把它读成一个明亮的瞬间，所以切换本身永远看不见——电影里藏剪切点用的也是同一个技巧。

## 什么时候用
- 遮住一次廉价或即时的内容切换，交叉淡入淡出会显得平淡的地方
- 高能量的品牌网站、产品发布和作品集画廊，想要冲击力
- 音乐、时尚和活动页面，漏光与视觉语言相符
- 任何你希望过渡本身读起来像一个刻意的节拍、而不只是管道的导航

## 工作原理
一个绝对定位的覆盖层以高 `z-index` 位于页面上方。过渡分三个阶段，由两次 `setTimeout` 驱动：把覆盖层在 `flashIn` 内淡入，在不透明度峰值处换掉当前页面，再在 `flashOut` 内把覆盖层淡回。内容变化发生在覆盖层不透明的时候，所以看不见：

```js
function doTransition(prev, next, flashIn, flashOut) {
  flashEl.style.transition = `opacity ${flashIn}ms ease-in, filter ${flashIn}ms ease`;
  flashEl.classList.toggle('blur-on', blurTog.checked);   // the optional glow
  flashEl.style.opacity = intensity;            // fade the flash up
  setTimeout(() => {
    // Peak — swap the page behind the opaque flash
    pages[prev].classList.remove('active');
    pages[next].classList.add('active');
    flashEl.style.transition = `opacity ${flashOut}ms ease-out, filter ${flashOut}ms ease`;
    flashEl.style.opacity = '0';               // fade the flash down to reveal
    flashEl.classList.remove('blur-on');
  }, flashIn);
}
```

淡出在与新不透明度相同的步骤里设置自己的过渡，所以它按 `flashOut` 而不是 `flashIn` 进行。无论按什么（页面名、「演示一下」或重置），上一次闪光开始后不到半秒不会开始新的闪光，所以演示每秒闪光不超过两次；更早提出的切换会等到那个时刻，覆盖层保持透明。闪光期间可选的 `blur()` 滤镜会柔化边缘，卖出漏光的感觉。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 闪光颜色 | 白色 | 白色读起来像相机闪光灯；金色、青色和红色读起来像彩色漏光 |
| 速度 | 普通 | 闪光升起和淡出有多快：慢是 320ms 升、510ms 降，普通 200ms 和 320ms，快 120ms 和 190ms；淡出总比升起稍长一点 |
| 闪光强度 | 全 | 闪光变得多实：全强度完全遮住页面切换；强（75%）和柔和（50%）会让它透出来 |
| 模糊闪光 | 关 | 把闪光的边缘柔化成辉光 |

## 生产注意事项
- **只在真正的峰值换页。**如果强度降到 ~0.9 以下，内容变化会透过覆盖层显现为一次硬切。保持峰值不透明度足够高，只在刻意要透视效果时才降低它。
- **防止重入。**一个 `animating` 标志会在过渡中途阻止新的导航；没有它，快速双击可能在覆盖层已经淡出时换页，露出接缝。
- **无障碍。**全屏白色闪光可能让人不适或诱发光敏反应。这个演示在 `prefers-reduced-motion: reduce` 下完全隐藏覆盖层、直接切换——生产环境也要这么做，并避免快速重复的闪光。
- **等价的库。**View Transitions API 可以用一个闪光色的伪元素给 `::view-transition-group` 做动画来复现这一点，不过专用的覆盖层更简单。GSAP 时间线能干净地串起淡入、换页回调和淡出。Barba.js 的 `leave`/`enter` 钩子直接对应峰值-揭示结构。

## 另见
- [交叉淡入淡出过渡](../crossfade/) — 它装扮起来的普通淡入淡出
- [溶解过渡](../dissolve/) — 瓷砖藏住切换
- [模糊过渡](../blur-transition/) — 模糊藏住切换
- [传送门 / 隧道缩放](../portal-zoom/) — 下一页从点击的圆圈里打开
