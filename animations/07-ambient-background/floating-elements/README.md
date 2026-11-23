# 漂浮元素

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/floating-elements/)

## 它是什么
漂浮元素是小的半透明图形，比如圆形、方形或圆环，在背景上慢慢漂移。每个图形沿着自己平滑、循环的路径走，有自己的速度、距离和起点，所以没有两个会同步移动。正是这种独立让它看起来像有生命的背景，而不是屏保。

## 什么时候用
- SaaS、金融科技和科技产品落地页的主视觉背景
- 应用引导画面，背景增添生气又不与界面抢注意力
- 空状态和加载画面，环境应该感觉“有人住”
- 仪表盘主视觉区，背景区分各个部分

## 工作原理
每个图形有一个基础位置（`bx`、`by`）和正弦波参数。每帧动画时，它的当前位置由基础位置加上运行时间 `t` 的正弦偏移算出：

```js
function animate(t) {
  elements.forEach(el => {
    const x = el.bx + Math.sin(el.freq * t + el.phase)    * el.amplitude;
    const y = el.by + Math.cos(el.freq * t * 0.7 + el.phaseY) * el.amplitude * 0.6;

    const rot = el.rotation ? el.rot : 0;   // its own small turn, added up in the loop below
    const opacity = el.pulse
      ? 0.4 + 0.3 * Math.sin(el.pulsePhase + t * 0.5)   // stays inside 0.1–0.7
      : 0.5;

    el.dom.style.transform = `translate(${x}px, ${y}px) rotate(${rot}deg)`;
    el.dom.style.opacity = opacity;
  });
}
```

时钟 `t` 和每个图形的转动按流逝的时间移动，而不是每帧固定量。每六十分之一秒给 `t` 加上 `speed * 0.005`（慢动作里是三分之一），不管需要多少帧，所以 30 Hz 的手机和 144 Hz 的显示器漂移速度相同。开始、暂停或从隐藏标签页返回后的第一帧什么也不加，帧之间的长间隔最多算 50ms：

```js
const FRAME = 1000 / 60;                  // what one 60 Hz frame adds up to
let last = null;                          // set back to null on Play and when the tab returns

function loop(now) {
  requestAnimationFrame(loop);
  const dt = last === null ? 0 : Math.min(now - last, 50);   // ms since the last frame
  last = now;
  const k = dt / FRAME * (slow ? 1 / 3 : 1);   // 1 at 60 Hz, 2 at 30 Hz, about 0.42 at 144 Hz; a third of that in slow motion
  t += speed * 0.005 * k;
  if (rotates) elements.forEach(el => { el.rot += el.rotSpeed * k; });
  animate(t);
}
```

**关键构造**——没有两个图形共享相同的参数：

```js
function createShape(count) {
  const amp = RANGE * 0.5 + Math.random() * RANGE * 0.5; // half to all of How far they drift (30, 50 or 90px)
  const freq = 0.3 + Math.random() * 0.5;                // its own pace, 0.3 to 0.8 times as fast as t
  const phase  = Math.random() * Math.PI * 2;            // random start phase
  const phaseY = Math.random() * Math.PI * 2;            // Y axis out-of-phase
  // Y multiplier (0.6) makes motion elliptical rather than circular
}
```

**图形生成**——不用图片资源，用代码定样式的图形：

```js
if (shape === 'ring') {
  el.style.borderRadius = '50%';
  el.style.border = '2px solid ' + color;   // outline only
  el.style.background = 'none';
} else if (shape === 'square') {
  el.style.background = color;
} else {
  el.style.borderRadius = '50%';            // circle
  el.style.background = color;
}
```

**鼠标排斥**——可选；图形轻轻地推离光标：

```js
if (mouse.x > 0) {
  const dx = x - mouse.x, dy = y - mouse.y;
  const d = Math.sqrt(dx*dx + dy*dy);
  if (d > 0 && d < 120) {
    x += (dx / d) * (120 - d) * 0.08;
    y += (dy / d) * (120 - d) * 0.08;
  }
}
```

指针位置从 `pointerdown` 和 `pointermove` 都会取，所以在触屏上，停在舞台上的手指也会推开图形。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 形状数量 | 中等 | 少是 5 个，中等 8 个，多是 14 个；5 到 12 个感觉平静，更多会忙乱 |
| 速度 | 普通 | 图形漂移的速度：每个图形左右来回摆动一次，慢速 87 到 233 秒，普通 52 到 140 秒，快速 33 到 87 秒，上下则要 1.4 倍时间 |
| 漂移距离 | 中等 | 每个图形离它位置游走多远：短是最多 30px，中等 50px，长是 90px |
| 形状 | 混合 | 圆形、方形、圆环或混合 |
| 颜色 | 冷色 | 冷蓝色和紫色、暖橙色和红色、灰色，或明亮的霓虹色 |
| 形状旋转 | 开 | 每个图形漂移时按自己的速度慢慢旋转 |
| 淡入淡出 | 开 | 每个图形慢慢变亮变暗，增加深度 |
| 形状避开指针 | 关 | 指针或手指 120px 内的图形滑到一边 |

## 生产注意事项
- **`requestAnimationFrame`，而不是 CSS `animation`**：每个元素单独的 `@keyframes` 需要生成独特的动画名。JS 循环对这种参数化做法更干净。
- **`will-change: transform`**：给每个漂浮元素加上它，提升到自己的合成层。12 个以上元素时，测一下这会减少还是增加 GPU 内存压力。
- **很多图形时避免 DOM 里的 SVG**：带复杂路径的 `<svg>` 元素比简单 CSS 样式的 `<div>` 合成起来更贵。坚持用 CSS 能实现的图形。
- **Framer Motion**：`<motion.div animate={{ x: [0, amplitude, 0, -amplitude, 0], y: [0, amplitude*0.6, 0, -amplitude*0.6, 0] }} transition={{ duration: 1/freq, repeat: Infinity, ease: "easeInOut" }} />`——每个元素设置独特的属性。
- **Particle.js / tsParticles**：用配置选项、碰撞检测和网络连线处理这个效果。生产环境需要的不只是基础漂浮时用它。

## 另见
- [Canvas 粒子效果](../../06-3d-advanced/canvas-particle-effect/) — 许多连起来、对指针有反应的粒子
- [环境涟漪](../ambient-ripple/) — 圆环扩散开来，而不是图形漂移
- [噪声驱动运动](../../06-3d-advanced/noise-based-motion/) — 平滑、自然的随机运动
