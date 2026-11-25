# ASCII / 半调

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/ascii-halftone/)

## 它是什么
ASCII 或半调效果用网格上的小标记重画一张图片。图片缩小到每个格子一个像素，每个格子的亮度挑一个字符（越亮的格子，墨越多的字符），或决定一个圆点的大小，就像报纸上的照片。因为图片每帧都会重新读取，所以它能用在任何动的东西上：Canvas 场景、视频或摄像头。

## 什么时候用
- 科技品牌和开发者工具的复古、终端和黑客风格外观
- 想要印刷或屏幕质感的音乐、游戏和活动页面
- 应该显得是做出来的、而不是拍出来的主视觉图片和作品集页眉
- 加载或闲置画面，动态图片在那里变成图案

## 工作原理
每帧，动态图片画在一块很小的隐藏画布上，每个格子一个像素，再读回来。这块画布用 `willReadFrequently` 创建，所以浏览器把它留在内存里，读取很便宜：

```js
const src = document.createElement('canvas');
const sctx = src.getContext('2d', { willReadFrequently: true });
src.width = cols; src.height = rows;                    // one pixel per cell
sctx.setTransform(dpr / cw, 0, 0, dpr / ch, 0, 0);       // draw the scene in stage pixels, shrunk to the grid
scene(sctx, gridWidth, gridHeight);
const px = sctx.getImageData(0, 0, cols, rows).data;

function level(px, i) {                                  // 0 to 1: how bright the cell is (how dark, on paper)
  const v = (0.2126 * px[i] + 0.7152 * px[i + 1] + 0.0722 * px[i + 2]) / 255;
  return paper ? 1 - v : v;
}
```

**每个标记一种图案。**用 `fillText` 或 `drawImage` 在每个格子里画一个字符，每个格子要一次调用，格子小时一帧要 9000 多次。取而代之，每个标记只在与一个格子等大的瓷砖上画一次，再做成重复图案。相邻且共用一个标记的格子连成一个矩形，每个标记的矩形一次填完，所以一帧大约只有十次填充：

```js
const RAMP = ' .-:*+=xo#';                               // from no ink to the most ink
pats = [...RAMP].map(c => {
  const tile = document.createElement('canvas'), g = tile.getContext('2d');
  tile.width = cw; tile.height = ch;                     // one cell, in device pixels
  g.font = `600 ${ch}px "Schibsted Grotesk"`; g.textAlign = 'center'; g.textBaseline = 'middle';
  g.fillStyle = '#fff'; g.fillText(c, cw / 2, ch * 0.55);
  return ctx.createPattern(tile, 'repeat');              // repeats every cell, so it lines up with the grid
});

const top = pats.length - 1, runs = pats.map(() => new Path2D());
for (let j = 0; j < rows; j++) {
  let from = 0, run = 0;
  for (let k = 0; k <= cols; k++) {
    const n = k < cols ? Math.round(level(px, (j * cols + k) * 4) * top) : 0;
    if (n === run) continue;
    if (run) runs[run].rect(from * cw, j * ch, (k - from) * cw, ch);
    from = k; run = n;
  }
}
for (let n = 1; n <= top; n++) { ctx.fillStyle = pats[n]; ctx.fill(runs[n]); }
```

圆点也是一样的办法：十二块瓷砖上的圆点，面积从没有均匀增大到填满格子，这正是半调看起来对的原因（眼睛读的是墨的占比，不是半径）。

**一步上色。**标记先以白色画在透明画布上。然后 `source-in` 只在已有标记的地方上色：用一种颜色，或者把那张小图片本身不经平滑地拉伸到整个网格上，这样每个标记都取到自己格子的颜色：

```js
ctx.globalCompositeOperation = 'source-in';
if (fullColor) { ctx.imageSmoothingEnabled = false; ctx.drawImage(src, 0, 0, cols * cw, rows * ch); }
else { ctx.fillStyle = ink; ctx.fillRect(0, 0, cvs.width, cvs.height); }
ctx.globalCompositeOperation = 'source-over';
```

字符用网站字体画。它的字形宽度并不都一样，所以字符阶梯只用比格子窄的字符（宽的 `@` 或 `%` 会在瓷砖边缘被切掉），按每个字符实测的墨量排序。场景的时钟按自上一帧以来的时间走，最多 50ms，慢动作里是它的三分之一。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 用什么画 | 字母 | 字母按亮度给每个格子十个字符中的一个，从空格到井号；圆点给它十二种圆点大小中的一个，从没有到填满格子，面积均匀递增 |
| 格子大小 | 中等 | 一个格子的高度：小是 8px，中等 11px，大是 16px，手机上用 10、14 和 20px；字母格子的宽是高的 0.7 倍，圆点格子是正方形 |
| 颜色 | 单色 | 单色把每个标记都涂成薄荷绿，浅色纸上是近黑色；全彩让每个标记取画面中自己位置的颜色 |
| 显示原图 | 关 | 在标记下面以 40% 的不透明度画出动态图片，这样能看出每个标记代表什么 |
| 浅色纸 | 关 | 浅色纸面配深色标记：每个格子的深浅（而不是亮度）决定标记，就像印出来的照片 |

## 生产注意事项
- **数调用次数，不数格子**：软件渲染测试里，格子小时每个格子一次 `drawImage` 一帧要 36ms；上面的图案填充画同一帧只要约 4ms，格子大小不限。
- **视频和摄像头**：每帧把 `<video>` 画进那块小画布，而不是画一个场景。来自其他网站的视频需要 CORS 头，否则读取像素会抛安全错误。
- **字形和字体**：等宽字体里每个字符宽度相同，任何字符阶梯都适用；比例字体要测量字符，只保留比格子窄的。
- **WebGL**：高分辨率下的全屏效果，用带字符纹理的片元着色器来做。three.js 自带 `AsciiEffect` 和 `DotScreenShader`；postprocessing 库有 ASCII 和圆点屏效果。
- **纯 CSS**：静止的半调可以是用 `radial-gradient` 做的圆点图案，再由图片遮罩，不用任何脚本。
- **无障碍**：标记会遮住图片的细节，所以把它的含义写进文字（alt 文本或 `aria-label`），效果本身只做装饰。
- **减弱动态**：显示一帧静止画面；演示开始时处于暂停状态，画着完整的图片。

## 另见
- [矩阵雨](../../07-ambient-background/matrix-rain/) — 落下的字符作为动态背景
- [悬停图片扭曲](../image-distortion-hover/) — 另一种在图片动起来时重塑它的办法
- [颗粒噪点叠加](../../07-ambient-background/grain-overlay/) — 铺在图片上的一层细腻质感
