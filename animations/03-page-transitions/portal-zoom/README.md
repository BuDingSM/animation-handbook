# 传送门 / 隧道缩放

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/portal-zoom/)

## 它是什么
传送门缩放把页面上的一个小元素当作通往下一页的窗口。点击它会从元素中心打开一个圆形（或方形），不断变大，直到新页面填满整个视图，感觉像潜入传送门，而不是跳到新页面。新页面本来就在下面；不断变大的开口只是揭示它。

## 什么时候用
- 图库或案例研究的条目，缩略图应该「打开进入」详情视图
- 空间导航的比喻——房间、世界、章节之间由门洞连接
- 落地页上一个值得隆重揭示的强行动号召
- 任何运动起点带有含义的过渡（这张卡片、这个按钮）

## 工作原理
进入的页面被裁成一个零半径的圆，位于传送门中心，设为激活、强制回流，然后过渡到一个大到能盖住舞台的圆。传送门的中心是相对舞台矩形来量的，所以揭示总是正好从用户点击的地方开始：

```js
const areaRect = area.getBoundingClientRect();
let px = '50%', py = '50%';
if (portalEl) {
  const pr = portalEl.getBoundingClientRect();
  px = ((pr.left + pr.width / 2 - areaRect.left) / areaRect.width * 100).toFixed(1) + '%';
  py = ((pr.top + pr.height / 2 - areaRect.top) / areaRect.height * 100).toFixed(1) + '%';
}
nextEl.style.clipPath = `circle(0% at ${px} ${py})`;   // pinhole at the portal
nextEl.style.transition = 'none';
nextEl.classList.add('active');
nextEl.offsetHeight;                                    // force reflow to commit start
nextEl.style.transition = `clip-path ${dur}ms ${ease}`;
nextEl.style.clipPath = `circle(150% at ${px} ${py})`;  // expand to cover the stage
```

结束半径是 `150%` 而不是 `100%`，因为圆必须到达矩形的角落，而角落离中心比边缘更远。在设置起始裁剪和动画过渡之间读一下 `offsetHeight`，会强制浏览器确认针孔状态，这样生长才从零开始做动画。`ease-in` 向外加速，读起来像放大；`ease-out` 读起来像缩小。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 传送门形状 | 圆形 | 圆形感觉像隧道；方形打开时像窗户 |
| 速度 | 普通 | 开口填满视图需要多长时间：慢是 1900ms，普通 1200ms，快 700ms；太快就变成普通的擦除 |
| 感觉 | 加速 | 加速感觉像潜入其中；平滑在结尾减速，像拉回来；轻柔先慢后快再慢；均匀保持一个稳定的速度 |

## 生产注意事项
- **强制回流。**在零半径裁剪和过渡之间不读 `offsetHeight`（或其他布局属性），浏览器会合并两次写入，页面直接以完整尺寸出现。回流读操作是让生长做动画的原因。
- **角落覆盖。**`circle(100%)` 会让四个角在动画中途一直没被裁剪。用约 150%，或者如果需要揭示在碰到最后一个像素时正好结束，就计算到最远角落的精确距离。
- **`clip-path` 合成。**给 `clip-path` 做动画对 GPU 友好，但在低端设备上、大尺寸时也不是免费的。让被裁剪元素的绘制区域保持合理，避免与重的模糊或阴影滤镜叠加。
- **等价的库。**View Transitions API 是最自然的选择——在 `::view-transition-new(root)` 上给 `clip-path` 做动画，从点击处的针孔开始，原生实现同样的效果。GSAP 直接给 `clipPath` 字符串做动画；Framer Motion 给 `clipPath` 样式属性做动画。这也是常见的着色器/WebGL 揭示方式，但 clip-path 不用画布就能覆盖 DOM 场景。

## 另见
- [缩放过渡](../zoom-transition/) — 整个页面缩放而不是打开
- [共享元素过渡](../shared-element-transition/) — 一张图片长进下一页
- [闪光 / 漏光过渡](../flash-transition/) — 一道闪光藏住切换
- [变形过渡](../morph-transition/) — 形状变化而不是移动
- [圆形揭示](../circle-reveal/) — 下一页从你点击的任意位置以圆形长大
