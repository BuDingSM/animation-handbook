# 噪声驱动运动

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/noise-based-motion/)

## 它是什么
噪声驱动运动用平滑噪声（一个随位置逐渐变化的随机值）让东西自然地动。普通的随机数会乱跳；噪声是漂移的，所以相邻的圆点动得相似，什么都不会猛地一抖。演示把它做成一片像风中草一样摇摆的圆点，和一个边缘起伏的色块。

## 什么时候用
- 有机背景元素：漂浮的圆点、起伏的色块、波动的网格
- 需要微妙的、非机械生命感的角色动画（待机呼吸、头发飘动）
- Canvas 或 WebGL 场景里程序化生成的地形和水面
- 任何应该感觉自然、而不是机械或随机的运动

## 工作原理
Simplex 噪声（Stefan Gustavson，公共领域）对任何 2D 或 3D 坐标返回 [-1, 1] 里的一个值。在 `(x * scale, y * scale + time)` 处采样噪声，就得到每个网格点上平滑的“风向”：

```js
function drawWindField() {
  for (let x = step/2; x < W; x += step) {
    for (let y = step/2; y < H; y += step) {
      const n = simplex2(x * SCALE, y * SCALE + time);
      const angle = n * Math.PI * 2;       // map [-1,1] to full circle
      const dx = Math.cos(angle) * amplitude;
      const dy = Math.sin(angle) * amplitude;

      ctx.beginPath();
      ctx.arc(x + dx, y + dy, 1.5, 0, Math.PI * 2);
      ctx.fill();
      // Draw a short line showing direction
      ctx.moveTo(x, y);
      ctx.lineTo(x + dx, y + dy);
      ctx.stroke();
    }
  }
  time += speed * 0.01 * (dt / FRAME);  // advance through noise volume
}
```

`dt` 是自上次绘制以来的时间（毫秒，封顶 50），`FRAME` 是 1000 / 60，即 60 Hz 屏幕上一帧的长度。所以噪声在 30、60 或 144 Hz 的屏幕上每秒移动同样的距离；每帧固定 `time += speed * 0.01` 的话，在 120 Hz 的屏幕上会移动两倍快，在 30 Hz 的手机上慢一半。

演示把 `step` 设为像素间距而不是列数，所以手机更小的舞台以同样的间距画更少的圆点。

**色块**——每个顶点的半径按它角度处采样的噪声偏移：

```js
ctx.beginPath();
for (let i = 0; i <= vertices; i++) {
  const a = (i / vertices) * Math.PI * 2;
  const n = simplex2(Math.cos(a) * 48 * SCALE + time,
                     Math.sin(a) * 48 * SCALE);
  const r = baseRadius + n * amplitude * 2;
  const x = cx + Math.cos(a) * r;
  const y = cy + Math.sin(a) * r;
  i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
}
ctx.closePath();
```

噪声沿半径为 `48 * SCALE` 的圆读取。很小的圆几乎只看到一个值，整条边缘只会一起胀大缩小；这个圆在默认尺寸下横跨约三个噪声凸起，所以边缘会起伏。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 形状 | 圆点阵 | 一片摇摆的圆点，或一个轮廓起伏的色块 |
| 图案大小 | 中等 | 噪声的图案有多大：噪声的一个漩涡在大时约 250px 宽，中等 125px，小 60px，色块边缘大约有两、三或五个起伏；大图案让大片区域一起摇摆，小图案显得热闹细腻 |
| 速度 | 普通 | 噪声移动的速度：图案漂过一个漩涡的宽度，慢约 5.6 秒，普通 3.3 秒，快 2.1 秒，中等图案下就是每秒约 22px、38px 和 60px |
| 晃动幅度 | 中等 | 每个圆点或色块边缘移动多远：小是 6px，中等 12px，大是 20px（离每个圆点的位置），色块边缘进出最多两倍 |
| 圆点间距 | 中等 | 圆点阵里圆点之间的距离：宽松是 64px，中等 48px，紧密 32px；更小的舞台画更少的圆点 |
| 颜色 | 蓝色 | 圆点或色块的颜色 |

## 生产注意事项
- **Simplex 与 Perlin**：Simplex 噪声（Gustavson 2005）比经典 Perlin 噪声更快，方向性瑕疵更少。新项目用 Simplex。
- **用 3D 噪声表示时间**：在 3D 里按 `(x, y, time)` 采样噪声，得到完全无缝的时间动画——图案永远不会“重置”。像演示这样用时间做偏移的 2D 噪声更简单，但在空间边缘可能有轻微的不连续。
- **`glsl-noise` / `open-simplex-noise`**：WebGL 着色器里内联一个 GLSL 噪声实现。JavaScript 里 `open-simplex-noise`（npm）是现代标准。
- **按时间，不按帧**：按自上一帧以来的时间穿过噪声，因为每帧固定一步会跟着屏幕的刷新率走。封顶这个时间（演示用 50ms），否则离开一分钟后回来的标签页会向前跳一分钟的运动，并让回来的标签页从下一帧开始。
- **流场**：风场变体就是“流场”——生成艺术里的经典手法（Daniel Shiffman 的 Coding Train）。粒子像溪流上的叶子一样跟随噪声场。

## 另见
- [Canvas 粒子效果](../canvas-particle-effect/) — 由简单物理而不是噪声移动的圆点
- [流场](../../07-ambient-background/flow-field/) — 粒子跟随噪声场并留下拖影
- [体积烟雾 / 3D 噪声](../volumetric-smoke/) — 同一种噪声，在 3D 里画成烟雾
