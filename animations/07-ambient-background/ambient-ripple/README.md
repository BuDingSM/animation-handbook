# 环境涟漪

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/ambient-ripple/)

## 它是什么
环境涟漪一次次从几个点发出圆环，像水滴落在静水面上，或声呐屏幕的回声。每个圆环从无到有，扩散时淡出，下一个在它消失前就开始了，所以圆环重叠成轻柔、稳定的脉动。它自己运行，不响应点击，暗示那里有东西活着。

## 什么时候用
- 主视觉区后面，“这里有东西活着”能强化产品的价值主张
- “已连接”或“活跃”状态的状态指示，需要有视觉存在感但不惹人警觉
- 冥想和专注应用，轻柔的径向脉动引导注意力
- 地图或基于位置的界面，脉动的圆环表示一个兴趣点

## 工作原理
每个涟漪源都有一个计时器，按可配置的间隔生成新的 `Ring` 对象。圆环每帧用 2D 画布更新和绘制：

```js
class Ring {
  constructor(x, y, now) {
    this.x = x;
    this.y = y;
    this.r = 0;
    this.born = now;   // the page's own clock, which stops while paused
    this.duration = RING_LIFE_MS;
  }

  update(now) {
    const progress = (now - this.born) / this.duration; // 0 → 1
    this.r = MAX_RADIUS * progress;
    this.opacity = (1 - progress) * 0.5;   // fade out as it expands
  }

  draw(ctx) {
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.r, 0, Math.PI * 2);
    ctx.strokeStyle = `rgba(88, 166, 255, ${this.opacity})`;
    ctx.lineWidth = THICKNESS * (1 - this.r / MAX_RADIUS * 0.3);
    ctx.stroke();
  }

  isDead(now) { return now - this.born > this.duration; }
}
```

`now` 是页面自己的时钟：每帧加上自上一帧以来的时间（慢动作里是三分之一），动画暂停时它停住，所以圆环原地定住、继续时不会跳变。

涟漪源以不规则的间隔发出圆环，避免机械的时钟感：

```js
function scheduleEmit(source) {
  const baseInterval = EMIT_INTERVAL_MS;
  const jitter = IRREGULAR ? (Math.random() - 0.5) * baseInterval * 0.8 : 0;
  source.nextEmit = clock + baseInterval + jitter;
}
```

**涟漪源漂移**——涟漪源在舞台上慢慢游走，增加有机的感觉：

```js
function updateSource(source) {
  source.x += source.vx * k;   // k is the time since the last frame in 60ths of a second: 1 at 60 Hz, 2 at 30 Hz, a third of it in slow motion
  source.y += source.vy * k;
  // Bounce at boundaries
  if (source.x < W * 0.05 || source.x > W * 0.95) source.vx *= -1;
  if (source.y < H * 0.1  || source.y > H * 0.9)  source.vy *= -1;
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 每个圆环扩散、淡出需要的时间：慢速 4 秒，普通 2.5 秒，快速 1.5 秒 |
| 涟漪间隔 | 中等 | 每个点多久发出一个圆环：短是每 1.8 秒，中等 3 秒，长 5 秒 |
| 涟漪大小 | 中等 | 每个圆环扩散多远：小是 100px，中等 160px，大是 250px；大圆环配慢速看起来更平静 |
| 涟漪源数量 | 3 | 一个点感觉很专注；三个铺满舞台；五个像水面上的雨 |
| 圆环粗细 | 中等 | 每个圆环的宽度：细是 1px，中等 1.5px，粗是 3px |
| 圆环颜色 | 蓝色 | 圆环的颜色 |
| 时间不均匀 | 开 | 每个间隔随机长出或短掉最多 40%，所以圆环永远不会像时钟一样滴答 |
| 涟漪源漂移 | 关 | 这些点在舞台上慢慢游走 |

## 生产注意事项
- **这不是点击涟漪**：点击涟漪（[点击涟漪](../../04-micro-interactions/click-ripple/)）响应用户输入、确认一次操作。环境涟漪是被动装饰性的——它自动、持续地发出，不需要任何用户交互。
- **圆环密度校准**：用默认值（三个源，每 3 秒一个圆环，每个存活 2.5 秒），每个源大多数时候最多只有一个圆环在外，所以屏幕上同时有两三个圆环，中间有短暂的安静。间隔更短或寿命更长会重叠更多圆环；单个、均匀间隔的圆环像时钟，太多则像发疯。
- **Canvas 与 SVG 与 CSS**：这里 canvas 最好，因为圆环数量可变、位置动态。15 个以上同时动态的元素的 SVG 动画会产生昂贵的 SMIL 计算。CSS `@keyframes` 不容易动态生成元素。
- **用 IntersectionObserver 暂停**：这些循环一直在运行——元素离开屏幕时一定要暂停，避免在长页面上耗电。
- **无障碍**：这个动画纯粹是装饰性的。在 `prefers-reduced-motion` 下暂停，必要时完全移除画布——不会丢失任何内容。
- **不需要库**：圆环不过是 2D 画布上每帧几条弧线；想要像水一样弯曲画面的涟漪，PixiJS 的 `DisplacementFilter` 或着色器才是常用工具，就像 three.js 的 GPGPU 水面例子那样。

## 另见
- [点击涟漪](../../04-micro-interactions/click-ripple/) — 响应点击的涟漪
- [呼吸光晕](../breathing-glow/) — 一个变大变小的光晕，而不是圆环
- [抽象几何动效](../abstract-geometric-motion/) — 它的圆环图案从中心扩散圆环
- [水面涟漪](../../06-3d-advanced/water-ripple/) — 手指激起的涟漪扩散开来，弯曲下面的画面
