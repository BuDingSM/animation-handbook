# 墨水扩散揭示

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/ink-spread-reveal/)

## 它是什么
墨水扩散揭示用墨在潮湿纸上扩散的方式揭开一张图片。图片开始时藏在一层素色的纸色图层下面，墨滴一个接一个落下，每滴扩散成一个边缘不平整的墨渍；墨渍长大、汇合，最终显示整张图片。每个墨渍开始时向外冲得很快，洇开时慢慢变慢，这让它读起来像墨水，而不是一个在变大的形状。

## 什么时候用
- 艺术、插画或作品集网站的开场图片
- 值得慢慢入场的精选艺术品或照片
- 故事、海报和活动页面的开场，手工感
- 绘画、书法或版画页面的章节开场

## 工作原理
图片坐在一个 SVG 遮罩下面。遮罩里每滴墨对应一个白色墨渍：白色显示图片，其他地方保持纸色。每个墨渍只做一次，由一个圆做成，它的半径随着几个慢波和几个小快波游走，所以边缘看起来不平整、带毛边；「飞溅」会在它周围加上甩出去的小墨滴。一个带种子的随机序列让每次重画都做出同样的墨渍。

```js
// One blot of radius 1: a circle whose radius wanders with six waves, the slow ones larger. wob is how far it may wander.
function unitBlot(rand, wob, dots) {
  const waves = [2, 3, 5, 9, 14, 23].map((n, i) => [n, (rand() * 2 - 1) / (i < 3 ? 1 : 2.6), rand() * 6.283]);
  const sum = waves.reduce((s, w) => s + Math.abs(w[1]), 0) || 1;
  waves.forEach(w => { w[1] *= wob * 1.8 / sum; });
  const pts = []; let min = 1;
  for (let i = 0; i < 96; i++) {
    const a = i / 96 * 6.283; let k = 1;
    for (const [n, amp, ph] of waves) k += amp * Math.sin(n * a + ph);
    min = Math.min(min, k); pts.push([Math.cos(a) * k, Math.sin(a) * k]);
  }
  const splash = [...Array(dots)].map(() => { const a = rand() * 6.283, d = 1.08 + rand() * .3; return [Math.cos(a) * d, Math.sin(a) * d, .022 + rand() * .04]; });
  return { pts, splash, min };
}
```

每个墨渍的完整尺寸都算过，让墨渍合起来盖住整张图片：图片的每个点都属于离它最近的那滴墨，那滴墨的墨渍做得足够大，能到达它最远的那个点，即使轮廓在那里凹进去（半径除以轮廓最小的到达距离）。墨渍然后从它落下的地方开始，通过缩放同一个形状长大，缓动先快后慢，每滴比前一滴晚一会儿：

```js
grows.forEach(el => el.animate([{ transform: 'scale(0)' }, { transform: 'scale(1)' }],
  { duration: 1800, delay: +el.dataset.i * 360, easing: 'cubic-bezier(.15,.7,.25,1)', fill: 'both' }));
```

```html
<svg viewBox="0 0 900 440">
  <defs><mask id="ink"><g transform="translate(216 185)"><g class="grow" data-i="0"><path fill="#fff" d="…"/></g></g></mask></defs>
  <rect class="paper" width="900" height="440"/>
  <g class="edge"><!-- the same blots a little larger, dark --></g>
  <g mask="url(#ink)"><!-- the picture --></g>
</svg>
```

每个墨渍的一个深色副本，大 4%，坐在图片下面，显示为紧贴在前面的一圈边。最后一个墨渍长完后，循环按住图片，把它淡回纸色，再播放一次。减弱动态时，整张图片改为淡入。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 墨落在哪 | 几滴 | 「一滴」落在中间；「几滴」是三滴依次落在图片上；「从角落开始」是三滴从左上到右下依次落下 |
| 速度 | 普通 | 盖住图片要多长时间：「慢」大约 3.6 秒（每个墨渍长 2.6 秒，墨滴间隔半秒），「普通」2.5 秒（1.8 秒，间隔三分之一秒），「快」1.5 秒（1.1 秒，间隔五分之一秒） |
| 边缘 | 粗糙 | 「平滑」长出圆形的圆；「粗糙」让每个轮廓游走大约十分之一个半径；「飞溅」在每个墨渍周围加九个小墨滴（手机上五个） |
| 深色墨边 | 开 | 每个墨渍的一个深色副本，稍大一点，显示为紧贴在前面的一圈边；关掉，图片显示时没有边 |

## 生产注意事项
- **遮罩会被重绘。**给 SVG 遮罩里的形状做动画，每帧都会重绘被遮罩的图片。对一张这个尺寸的图片来说没问题；慢手机上的全屏主视觉，把墨渍画在 canvas 上，用它作 `mask-image`，或者直接揭示图片的一个 canvas 副本。
- **缩放，永远不要重画。**每个墨渍的轮廓只建一次，用 `transform: scale()` 让它长大。每帧重写路径会让浏览器重新解析、重新布局。
- **盖住角落。**中间看起来够大的墨渍常常会把角落留成纸色。按每个墨渍必须到达的点来定尺寸，或者让最后一滴最大。
- **减弱动态。**改为把图片淡入，而不是让墨渍长大。
- **GSAP** 用 `gsap.fromTo(".grow", { scale: 0 }, { scale: 1, duration: 1.8, ease: "power3.out", stagger: 0.36, transformOrigin: "0 0" })` 让墨渍长大；设置原点，让每个从它落下的地方长大。**Framer Motion** 给 `<mask>` 里的每个墨渍 `<motion.path>` 做动画，从 `scale: 0` 到 `scale: 1`，`transition={{ delay: i * 0.36, ease: [0.15, 0.7, 0.25, 1] }}`。
- **真墨水。**扫描的墨渍或一段短的墨水视频用作亮度遮罩，看起来最丰富；保持它小一点，画出来的墨渍作后备。

## 另见
- [剪裁路径揭示](../clip-path-reveal/) — 一个干净的形状长大，揭开一张图片
- [圆形揭示](../../03-page-transitions/circle-reveal/) — 一个圆从点击处长大，显示下一页
- [幕布揭示](../curtain-reveal/) — 一块彩色面板盖住它，然后滑走
- [淡入淡出](../fade-in-out/) — 减弱动态时这个效果变成的单纯淡入淡出
