# 星空

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/starfield/)

## 它是什么
星空在深色背景上布满小星星，从中心向外飞流，像从飞船上看太空。每颗星星的大小、亮度和速度取决于它离中间多远：中心附近的星星小、暗、慢，向边缘变大、变亮、变快，这正是它感觉深邃的原因。横向版本让星星飘过，越近的越快，像从窗户看出去。

## 什么时候用
- 太空、天文或科幻主题的应用
- 自助终端屏幕或专注应用的氛围屏保式背景
- 科技产品的主视觉区，“无限规模”或“超越地平线”就是信息
- 需要深度和运动感、又不要复杂的深色主题落地页

## 工作原理
每颗星星跟踪它的角位置（`angle`）、离中心的距离（`dist`）和速度。每帧，距离按速度乘以 `k`（下面解释）增加，星星的画布坐标由极坐标算出：

```js
class Star {
  constructor(W, H) {
    this.angle = Math.random() * Math.PI * 2;
    this.dist  = Math.random() * Math.max(W, H) * 0.5; // random start in field
    this.speed = (Math.random() * 0.6 + 0.2) * BASE_SPEED;
    this.size  = Math.random() * 1.5 + 0.5;
  }

  update(k) {
    this.dist += this.speed * k;
    const ratio = this.dist / MAX_DIST;
    // Accelerate as the star "approaches" — perspective foreshortening
    this.speed = (ratio * 0.5 + 0.2) * BASE_SPEED * 1.5; // faster toward the edge
    // Reset to center when off-screen
    if (this.dist > MAX_DIST) {
      this.dist = 0;
      this.speed = (Math.random() * 0.6 + 0.2) * BASE_SPEED;
      this.angle = Math.random() * Math.PI * 2;
    }
  }

  draw(ctx, cx, cy) {
    const ratio = this.dist / MAX_DIST;
    const x = cx + Math.cos(this.angle) * this.dist;
    const y = cy + Math.sin(this.angle) * this.dist;
    const size = this.size * (1 + ratio * 1.5);   // grow with distance
    const opacity = 0.3 + ratio * 0.7;            // brighten with distance

    ctx.fillStyle = `rgba(255, 255, 255, ${opacity})`;
    ctx.beginPath();
    ctx.arc(x, y, size, 0, Math.PI * 2);
    ctx.fill();
  }
}
```

**横向漂移变体**——星星有一个固定的深度值，同时决定大小和速度：

```js
class DriftStar {
  constructor(W, H) {
    this.x     = Math.random() * W;
    this.y     = Math.random() * H;
    this.depth = Math.random() * 0.8 + 0.2;  // 0.2 (far) to 1.0 (near)
    this.size  = this.depth * 2;
  }
  update(k) { this.x -= this.depth * SPEED * 2 * k; if (this.x < 0) this.x = W; }
}
```

`n` 是自上一帧绘制以来的 60 Hz 帧数，所以 30 Hz 的手机和 144 Hz 的显示器显示相同速度。开始、暂停或从隐藏标签页返回后的第一帧什么也不加，帧之间的长间隔最多算 50ms。星星按 `k` 移动，`k` 就是 `n`，慢动作里是它的三分之一：

```js
const n = Math.min(now - last, 50) / (1000 / 60);   // 60ths of a second since the last drawn frame: 1 at 60 Hz, 2 at 30 Hz
const k = n * (slow ? 1 / 3 : 1);                    // what update(k) gets
```

短拖尾来自用透明黑色层盖住画布而不是清空它。这层永远是 60 Hz 一帧的 0.85，每六十分之一秒盖一次（30 Hz 的帧盖两次），所以拖尾在每个屏幕上持续的秒数相同。慢动作不会让它们变慢。离整数 5% 以内的 `n` 在覆盖时算作那个整数，所以 59.94 或 60.06 Hz 的屏幕，或时间戳里的一点噪点，每帧仍然恰好盖一次，`fa` 带着剩下的小数部分。四舍五入向上偏 0.05，所以帧正好落在两次覆盖中间的屏幕，比如 120 或 240 Hz，能保持规律的绘制节奏，而不是由时间戳噪点决定。还没有整次覆盖到期时，比如 120 Hz 屏幕两帧中的第一帧，那一帧什么也不画，它的时间带到下一帧。这样无论什么屏幕，绘制最多大约每秒 60 帧（30 Hz 的屏幕画 30 帧，每帧盖两次），每帧绘制都盖一次：

```js
const w = Math.max(1, Math.round(n));
const f = fa + (Math.abs(n - w) < 0.05 ? w : n);   // n within 5% of a whole number counts as that number
const nf = Math.round(f + 0.05);                    // whole covers due; the 0.05 keeps a half-way tie off the rounding edge
if (!nf) return;                                    // none yet: draw nothing, the time carries on
fa = f - nf; last = now;                            // the fraction left over carries too
for (let i = 0; i < nf; i++) {
  ctx.fillStyle = 'rgba(0,0,0,0.85)'; ctx.fillRect(0, 0, W, H);
  if (haze) drawHaze();   // the nebula haze goes on after each cover, as it does at 60 Hz
}
```

一次更强的覆盖 `1 - 0.15 ** n` 理论上淡出的量相同，但 8 位画布每次覆盖都会四舍五入，所以淡淡的残留，比如薄雾，会在不同屏幕上沉淀成不同颜色。重复同样的覆盖让它们在所有地方沉淀成相同的颜色。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 方向 | 向外 | 向外让星星从中心飞流；横向让它们飘过，越近的越快 |
| 速度 | 普通 | 星星移动的速度：向外时，边缘附近的星星慢速、普通和快速每秒分别覆盖 16、25 和 41 像素，中心附近不到那个数的三分之一；横向时，最近的星星每秒覆盖 30、48 和 78 像素，最远的是那个数的五分之一；想要平静的背景就保持慢速 |
| 星星数量 | 中等 | 少是 150 颗，中等 300 颗，多是 600 颗；手机上最多显示 300 颗 |
| 星星颜色 | 白色 | 白色、暖白色，或每颗星星不同的淡色，从粉色到黄色 |
| 闪烁 | 开 | 每颗星星按自己的节奏轻轻变亮变暗 |
| 星云薄雾 | 关 | 星星后面淡淡的紫色和蓝色薄雾增加深度 |

## 生产注意事项
- **画布与 DOM**：星星超过 50 颗时 DOM 元素会造成大量布局重算。画布是这个效果的正确工具。
- **手机**：演示在手机尺寸的屏幕（宽最多 600px，或横持手机时高最多 500px）上最多画 300 颗星星；每颗星星是一次单独填充，所以数量是主要成本。
- **缓存 `ctx.fillStyle`**：每颗星星都设置 `fillStyle` 很贵。按不透明度分桶给星星分组，每桶设置一次 fillStyle（颜色批量），减少画布状态变化。
- **每个屏幕上速度相同**：每 `requestAnimationFrame` 帧固定步长，在 120Hz 显示器上跑两倍快，在 30Hz 手机上跑一半快。演示按自上一帧绘制以来的时间（以六十分之一秒计，最多 50ms）缩放每步，所以星星在所有地方以相同速度移动，而且只在整六十分之一秒到期时才画，在任何显示器上最多大约每秒 60 次，所以快速显示器不做额外工作。
- **搭配星云背景**：在星星后面加一个微妙的径向渐变（一个象限深紫色、另一个深蓝色），以最小的性能代价大幅提升真实感。
- **Three.js 的 `Points` 几何**：生产环境的星空用 Three.js 的 `BufferGeometry` 加 `PointsMaterial`。每颗星星是一个顶点；位置缓冲每帧更新。这种做法能扩展到 10 万颗以上的星星。
- **`prefers-reduced-motion`**：星星保持静止（没有动画循环），或限制为正常速度 10% 的非常缓慢的漂移。

## 另见
- [极光](../aurora/) — 与夜空搭配的光带
- [Canvas 粒子效果](../../06-3d-advanced/canvas-particle-effect/) — 连起来、对指针有反应的粒子
- [漂浮元素](../floating-elements/) — 沿自己路径慢慢漂移的图形
- [雪 / 雨](../snow-rain/) — 落下的雪或雨，而不是向你飞流
