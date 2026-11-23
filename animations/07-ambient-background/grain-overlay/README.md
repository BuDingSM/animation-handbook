# 颗粒噪点叠加

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/grain-overlay/)

## 它是什么
胶片颗粒在设计上铺一层半透明的随机圆点，一遍遍重画，让质感像摄影胶片的颗粒一样闪烁。强度低时，大约 5% 到 10%，几乎看不见，但扁平的数码色彩感觉更暖、更深；强得多时，看起来像坏掉的屏幕。诀窍是微妙：人们不应该注意到颗粒，只在它消失时想念它。

## 什么时候用
- 想要摄影、编辑美学的主视觉区和作品集页面
- 视频播放器叠加层，颗粒弥合数码与电影之间的差距
- 品牌标识很重的页面，“模拟暖意”是定位的一部分
- 深色主题仪表盘，纯扁平的表面感觉太冷

## 工作原理
**画布做法**——每帧生成一张新的随机噪点纹理（或以较低频率生成，获得像胶片一样的节奏）：

```js
function drawGrain(canvas, ctx, pixelSize = 2, colorNoise = false) {
  const { width: W, height: H } = canvas;
  const img = ctx.createImageData(W, H);
  const data = img.data;

  for (let y = 0; y < H; y += pixelSize) {
    for (let x = 0; x < W; x += pixelSize) {
      // one color per grain dot: gray, or three random channels
      const v = Math.random() * 255 | 0;
      const r = colorNoise ? Math.random() * 255 | 0 : v;
      const g = colorNoise ? Math.random() * 255 | 0 : v;
      const b = colorNoise ? Math.random() * 255 | 0 : v;
      for (let dy = 0; dy < pixelSize && y + dy < H; dy++) {
        for (let dx = 0; dx < pixelSize && x + dx < W; dx++) {
          const idx = ((y + dy) * W + (x + dx)) * 4;
          data[idx] = r; data[idx + 1] = g; data[idx + 2] = b; data[idx + 3] = 255;
        }
      }
    }
  }
  ctx.putImageData(img, 0, 0);
}

// Regenerate at 24fps (film rate)
let lastFrame = 0;
function loop(ts) {
  if (ts - lastFrame > 1000 / 24) {
    drawGrain(canvas, ctx);
    lastFrame = ts;
  }
  requestAnimationFrame(loop);
}
```

**SVG `feTurbulence` 做法**——浏览器原生生成噪点；循环 `seed` 属性就能让它动起来：

```html
<svg width="0" height="0" style="position:absolute">
  <defs>
    <filter id="grain" color-interpolation-filters="sRGB">
      <feTurbulence type="fractalNoise" baseFrequency="0.65" numOctaves="3"
                    stitchTiles="stitch" id="turb"/>
      <feColorMatrix type="saturate" values="0"/>
      <feComponentTransfer>
        <feFuncR type="linear" slope="3" intercept="-1"/>
        <feFuncG type="linear" slope="3" intercept="-1"/>
        <feFuncB type="linear" slope="3" intercept="-1"/>
        <feFuncA type="linear" slope="0" intercept="1"/>
      </feComponentTransfer>
    </filter>
  </defs>
</svg>

<div style="
  position: absolute; inset: 0;
  filter: url(#grain);
  opacity: 0.08;
  mix-blend-mode: overlay;
"></div>
```

原始噪点是淡淡的半透明灰色；分量转换让它不透明、拉伸对比度，所以它在相同不透明度下和画出的圆点一样明显。

```js
let seed = 0;
function loop() {
  document.getElementById('turb').setAttribute('seed', seed++);
  requestAnimationFrame(loop);
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 强度 | 轻微 | 颗粒有多明显：微弱是 4%，轻微 8%，强烈 16%；5–10% 是感觉到的多于看到的，超过 20% 看起来像坏掉的屏幕 |
| 颗粒大小 | 中等 | 每个圆点有多大：细是 1px，中等 2px，粗是 3px；粗的像老胶片 |
| 变化频率 | 像胶片 | 新颗粒每帧一次（每秒 60 次）、像胶片（24 次）、卡顿（12 次）或慢（4 次）；手机上每秒最多换 24 次 |
| 混合方式 | 胶片 | 颗粒如何与画面混合：胶片让暗部更暗、亮部更亮，像真实颗粒（叠加混合）；只提亮是仅提亮（滤色）；更柔和是更温和的胶片（柔光） |
| 彩色颗粒 | 关 | 随机彩色的圆点，而不是灰色的 |
| 颗粒来源 | 随机圆点 | 脚本画的圆点，或浏览器自己的噪点滤镜；颗粒大小和彩色颗粒只适用于圆点 |

## 生产注意事项
- **Canvas 与 SVG feTurbulence**：canvas 控制更多（像素大小、彩色噪点），但更耗 CPU。SVG feTurbulence 是 GPU 加速的、更简单，但对颗粒特性的控制更少。
- **降低更新频率是故意的**：真实胶片颗粒是 24fps，不是 60fps。每秒生成 60 次新的画布纹理是浪费算力——12–24fps 符合美学，还降低 CPU 负载。
- **手机**：演示在手机尺寸的屏幕（宽最多 600px，或横持手机时高最多 500px）上每秒最多换 24 次颗粒，并按 CSS 像素而不是设备像素画画布；画每个圆点才是耗费的部分。
- **`mix-blend-mode: overlay`** 是颗粒的标准做法：它让暗部稍暗、亮部稍亮，符合卤化银对曝光的响应。
- **伪元素上的 CSS 滤镜**：最干净的生产做法——加上 `::after { content:''; position:absolute; inset:0; background:url(grain.png); animation:grain 0.5s steps(1) infinite; }`，配一张预生成的颗粒帧精灵图。这样颗粒生成完全交给静态资源。
- **React 库**：`react-noise` 和各种 `css-grain` 包把 SVG 滤镜做法实现为零配置的即插即用组件。

## 另见
- [扫描线效果](../scanline/) — 页面上的深色线条，像老显示器
- [色差](../../06-3d-advanced/chromatic-aberration/) — 颜色在边缘分裂，像廉价镜头
- [漏光](../light-leak/) — 暖光漫入，像胶片相机的缺陷
