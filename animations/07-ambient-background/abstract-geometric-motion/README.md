# 抽象几何动效

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/abstract-geometric-motion/)

## 它是什么
抽象几何动效是由简单图形组成的无尽循环，纯粹为了观看而存在：多边形围绕中心慢慢旋转，圆环从中心扩散，斜条滑过，或波浪线带着色彩流动。没有故事，也没有什么可点击的。像屏保或音乐可视化一样，它就是要长时间保持平静、赏心悦目。

## 什么时候用
- “正在播放”和音乐播放器背景，画面应该随内容起伏
- 专注和冥想应用的背景，运动引导注意力向内
- 数字艺术装置和自助终端屏幕，屏幕绝不能显示静态画面
- 空状态和加载画面，环境应该感觉是设计过的，而不是空白的

## 工作原理
四种预设都在一个共享的 `requestAnimationFrame` 循环里运行，用一个全局时间计数器 `t`。每过去六十分之一秒，就给它加上 `speed * 0.01`，每个圆环长大 `speed * 2` 像素，不管需要多少帧，所以 120 Hz 的屏幕或掉帧都不会改变速度。每种预设都渲染到一个 `<canvas>` 元素上。

**旋转的多边形**——顶点数递增的正多边形，每个以略微不同的速度旋转：

```js
function drawPolygons(t) {
  for (let i = 0; i < shapeCount; i++) {
    const sides = 3 + (i % 5);           // 3 to 7 sides
    const radius = 30 + i * 20;
    const rotation = t * (0.2 + i * 0.08) + (i % 2 === 0 ? 0 : Math.PI);

    ctx.beginPath();
    for (let j = 0; j <= sides; j++) {
      const a = j / sides * Math.PI * 2 + rotation;
      ctx.lineTo(cx + Math.cos(a) * radius, cy + Math.sin(a) * radius);
    }
    ctx.strokeStyle = palette(i, 0.5 + 0.3 * Math.sin(t * 0.5 + i));
    ctx.lineWidth = lineWidth * (1 + i * 0.1);   // 0.8, or 1.5 with Thicker lines
    ctx.stroke();
  }
}
```

**同心扩散的圆环**——圆环从中心生成、向外扩散、淡出：

```js
// Spawn a new ring on a fixed interval of t, so spacing stays even at any speed
if (t - lastSpawn >= 0.5) { lastSpawn = t; rings.push({ r: 0, color: randomColor() }); }

// Update and draw each ring
rings.forEach(ring => {
  ring.r += speed * 2;
  const life = 1 - ring.r / Math.max(W, H);
  ctx.strokeStyle = `rgba(..., ${life * 0.6})`;
  ctx.arc(cx, cy, ring.r, 0, Math.PI * 2);
  ctx.stroke();
});
rings = rings.filter(r => r.r < Math.max(W, H));
```

选择圆环时，一开始就有六个圆环已经铺在舞台上，所以图案立刻就能看到，而不是从空舞台慢慢长出来。

**色彩流动的线**——正弦波曲线，用动态的 `lineDashOffset` 让颜色看起来沿着路径流动：

```js
ctx.setLineDash([40, 60]);
ctx.lineDashOffset = -dashPhase + i * 20;  // dashPhase grows by 30 * speed each time t grows by 1
ctx.strokeStyle = palette(i, opacity);
ctx.beginPath();
// ... draw sine wave path ...
ctx.stroke();
ctx.setLineDash([]);
```

**滑动的条形和流动的虚线**有自己的距离计数器 `barX` 和 `dashPhase`，`t` 每增长 1，它们分别增长 `40 * speed` 和 `30 * speed`。用累加计数器而不是把 `t` 乘以速度，意味着改变速度永远不会让条形或虚线跳变。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 图案 | 多边形 | 旋转的多边形、从中心扩散的圆环、滑动的条形，或带着流动色彩的波浪线 |
| 速度 | 普通 | 整体移动的速度：最里面的多边形慢速约 150 秒转一圈，普通 87 秒，快速 52 秒（外面的更快），圆环每秒扩散 42、72 或 120px，条形每秒滑动 3、9 或 24px；放在内容后面时避免快速 |
| 形状数量 | 中等 | 多边形、条形或线条有多少：少是 3 个，中等 6 个，多是 10 个（线条图案画两倍）；在圆环图案里，它决定圆环用多少种颜色 |
| 颜色 | 冷色 | 冷蓝色、暖琥珀色、明亮的霓虹色或灰色 |
| 更粗的线条 | 关 | 线条画成 1.5px 宽而不是 0.8px，图形更醒目 |

## 生产注意事项
- **Canvas 与 SVG 与 CSS**：canvas 最适合每帧都变的复杂动态几何图形。SVG SMIL 动画对少量元素有效，但独立动态路径多了就很耗性能。CSS 不适合运行时生成的几何图形。
- **`t += speed * 0.01`，而不是 `Date.now()`**：相对时间增量（累加计数器）本质上与帧率无关，也比绝对时间戳更容易控制。想要真正与帧率无关的运动，就乘以实际的帧间隔。
- **无限无缝循环**：什么都不会重启动。每种预设都从只会增长的值（`t`，以及条形和虚线自己的计数器）画起，条形用 `%` 绕过舞台，所以没有重置时刻。
- **用 `lineDashOffset` 实现色彩流动**：给 `lineDashOffset` 做动画是经典的 SVG/canvas 技巧，可以画出看起来有色彩沿长度流动或运动的路径。虚线图案固定在路径的局部坐标空间里；偏移量移动的是起点。
- **搭配音乐可视化**：把基于时间的 `t` 换成 Web Audio API 的 `AnalyserNode` 提供的音频频率数据。图形就会随音频频谱起伏、改变大小。
- **Three.js 等价实现**：`LineSegments`、`MeshLine` 和自定义 `ShaderMaterial` 可以在 3D 场景中重现全部四种预设，相机移动增加了第三个维度。

## 另见
- [环境涟漪](../ambient-ripple/) — 圆环从几个点自己扩散开来
- [WebGL 着色器动画](../../06-3d-advanced/webgl-shader-animation/) — 由显卡绘制的图案
- [噪声驱动运动](../../06-3d-advanced/noise-based-motion/) — 平滑、自然的随机运动
- [音频可视化](../../06-3d-advanced/audio-visualizer/) — 随音乐舞动的条形，从低音到高音
