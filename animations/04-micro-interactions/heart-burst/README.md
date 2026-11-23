# 点赞爱心迸发

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/heart-burst/)

## 它是什么
一个点赞按钮，在你点赞的瞬间庆祝一下。爱心迅速压扁、过头、再落定，同时填满颜色；小爱心和圆点从它的中心迸出来，淡出。取消点赞只是去掉颜色，没有庆祝。

## 什么时候用
- 信息流、画廊和评论区里的点赞 / 收藏 / 表情按钮
- 任何一个值得一点小小多巴胺奖励的、单一的二元“积极”操作
- 收藏或书签按钮，你想让保存的感觉来之不易
- 不适合有破坏性或中性的开关——庆祝意味着赞同

## 工作原理
爱心本身是一个内联 SVG，填充颜色随 `.on` 类过渡，再加一段关键帧弹出做过头效果。迸发是一个粒子系统，画在覆盖整个舞台的 `<canvas>` 上。点赞时，你在爱心的中心生成 N 个粒子，每个随机角度和速度；每一帧按过去的时间推进它们，施加重力和一点侧向阻力，生命值递减直到淡出。每一步都乘以 `k`，即按 60 Hz 帧衡量的、距离上一帧的时间：60 Hz 屏幕上是 1，120 Hz 上是 0.5，30 Hz 上是 2，所以迸发在所有屏幕上都持续大约四分之三秒：

```js
const FRAME = 1000 / 60;   // one frame of a 60 Hz screen, in ms
let last = null;           // time of the previous frame; null until a burst's first frame

function spawn() {
  const b = btn.getBoundingClientRect(), s = stage.getBoundingClientRect();
  const ox = b.left + b.width/2 - s.left, oy = b.top + b.height/2 - s.top;
  for (let i = 0; i < pCount; i++) {
    const a = Math.random() * Math.PI * 2, v = spread * (0.5 + Math.random()*0.5) / 12;
    particles.push({ x: ox, y: oy, vx: Math.cos(a)*v, vy: Math.sin(a)*v - 1, life: 1, /* ... */ });
  }
  if (!raf) { last = null; raf = requestAnimationFrame(tick); }
}

function tick(ts) {
  const dt = last === null ? FRAME : Math.min(ts - last, 50);   // ms since the last frame, at most 50
  last = ts;
  const k = dt / FRAME;
  ctx.clearRect(0, 0, cv.width, cv.height);
  particles = particles.filter(p => p.life > 0);
  for (const p of particles) {
    p.x += p.vx * k; p.y += p.vy * k; p.vy += 0.12 * k;    // move, gravity
    p.vx *= 0.98 ** k; p.life -= 0.022 * k;                // drag, fade
    ctx.globalAlpha = p.life;
    /* draw heart or dot at p.x, p.y */
  }
  raf = particles.length ? requestAnimationFrame(tick) : null;
}
```

一次迸发的第一帧算作一帧 60 Hz 的帧，超过 50ms 的间隔（标签页从后台回来）算作 50ms，所以什么都不会跳。演示还把超过 1.5 帧的一步（比如在 30 Hz 屏幕上，或掉帧之后）分成两半走，这样弧线和 60 Hz 屏幕上画的保持接近；慢动作会把 `k` 乘以三分之一。

初始 `vy` 上的 `-1` 偏置，让迸发在重力把它拉下来之前先向上倾斜。弹出通过去掉再重新加上动画类来重新触发（`btn.classList.remove('pop'); void btn.offsetWidth; btn.classList.add('pop')`），这样重复点赞总会重新触发。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 爱心数量 | 一些 | 迸出多少块碎片：少是 10 个，一些是 18 个，多是 32 个；超过大约 40 个会显得忙乱，还耗费帧 |
| 飞多远 | 中等 | 碎片在淡出前飞多远：短、中、远分别以最高 300、450 或 700px 每秒的速度发射（至少一半），大约四分之三秒内减速、落下、淡出，最快的最后落在爱心侧面大约 150、225 或 350px 处 |

## 生产注意事项
- **Canvas 而不是 DOM 粒子**：每次点按生成 18–40 个短命的 DOM `<span>`，用户连点时会产生布局/GC 抖动。一个 canvas 加一个普通对象数组，始终平稳又便宜。
- **按时间，不按帧**：把每一帧的量按距离上一帧的时间缩放，阻力系数也要升到同样的幂（`0.98 ** k`）。没有它，迸发在 120 Hz 屏幕上只持续一半时间，在 30 Hz 手机上持续两倍，围绕它计时的东西，比如演示里的“演示一下”，都会错位。
- **让循环闲下来**：`particles.length === 0` 时停掉 `requestAnimationFrame`，下一次生成时再启动。一个一直运行的 rAF，会给一个偶尔才触发的控件浪费电池。
- **DPR 缩放**：按 `devicePixelRatio`（上限约 2）设置 canvas 后备存储的大小，再 `ctx.setTransform(dpr,0,0,dpr,0,0)`，这样爱心在视网膜屏上保持清晰，又不会在 3x 手机上过度分配。
- **减弱动态效果**：完全跳过弹出和迸发——只切换填充颜色。状态变化仍然必须传达，只是庆祝被去掉。
- **无障碍**：用 `aria-pressed` 支持按钮，并随状态切换它；视觉上的迸发是装饰性的，应该 `aria-hidden`。
- **canvas-confetti**：想要现成的放射状迸发，`confetti({ particleCount, spread, origin })` 不用手写循环就能给你同样的物理效果。
- **Framer Motion / Lottie**：Framer 的 `AnimatePresence` 可以在数量少时驱动 DOM 粒子；很多生产应用则会改用预渲染的 Lottie 迸发，让各平台的画面像素一致。

## 另见
- [按钮按压缩放](../button-press-scale/) — 爱心弹出背后的压扁和回弹
- [通知徽标脉冲](../badge-pulse/) — 徽标脉冲，吸引注意
- [对勾绘制](../checkmark-draw/) — 不用迸发的成功表示
- [点击涟漪](../click-ripple/) — 从你按下的位置传来的、更平静的回应
