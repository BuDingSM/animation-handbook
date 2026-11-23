# 成功彩带

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/success-confetti/)

## 它是什么
成功彩带是一阵彩带，标记一个完成的任务，比如「下单」。按钮变成成功状态，很多彩色小矩形从它上面以扇形喷出，旋转、落下、淡出。它是一种奖励：它庆祝这一刻，但不增加新信息。

## 什么时候用
- 结账 / “下单”确认、注册完成、套餐升级
- 多步流程的最后一步，回报值得这番 flourish
- 学习或健身应用里的成就和连续打卡时刻
- 每个事件一次，真正的里程碑——每次点击都放彩带，会训练用户无视它

## 工作原理
一个 `<canvas>` 覆盖层盖住舞台。点击时，你在按钮中心生成一批彩带碎片，每块按可配置的扩散角度，围绕笔直向上的基准方向扇开。每一帧对 `vy` 施加重力，推进位置，旋转碎片，消耗生命；碎片淡出或离开屏幕后退役。每一步都乘以 `k`，即按 60 Hz 帧衡量的、距离上一帧的时间：60 Hz 屏幕上是 1，120 Hz 上是 0.5，30 Hz 上是 2，所以一块碎片在所有屏幕上都在大约两秒内淡出：

```js
const FRAME = 1000 / 60;   // one frame of a 60 Hz screen, in ms
let last = null;           // time of the previous frame; null until a burst's first frame

function fire() {
  const half = (spreadDeg * Math.PI/180) / 2;
  for (let i = 0; i < pCount; i++) {
    const ang = -Math.PI/2 + (Math.random()*2 - 1) * half;   // fan around straight up
    const speed = 6 + Math.random()*7;
    particles.push({ x: ox, y: cy, vx: Math.cos(ang)*speed, vy: Math.sin(ang)*speed,
      w: 6+Math.random()*6, h: 3+Math.random()*5, rot: Math.random()*Math.PI,
      spin: (Math.random()-0.5)*0.3, color: COLORS[i % COLORS.length], life: 1 });
  }
  if (!raf) { last = null; raf = requestAnimationFrame(tick); }
}

function tick(ts) {
  const dt = last === null ? FRAME : Math.min(ts - last, 50);   // ms since the last frame, at most 50
  last = ts;
  const k = dt / FRAME;
  ctx.clearRect(0, 0, cv.width, cv.height);
  particles = particles.filter(p => p.life > 0 && p.y < stage.clientHeight + 40);
  for (const p of particles) {
    p.vy += gravity * k; p.vx *= 0.99 ** k;                  // gravity, drag
    p.x += p.vx * k; p.y += p.vy * k; p.rot += p.spin * k;   // move, spin
    p.life -= 0.008 * k;                                     // fade
    ctx.save(); ctx.translate(p.x, p.y); ctx.rotate(p.rot);
    ctx.globalAlpha = p.life; ctx.fillStyle = p.color;
    ctx.fillRect(-p.w/2, -p.h/2, p.w, p.h);
    ctx.restore();
  }
  raf = particles.length ? requestAnimationFrame(tick) : null;
}
```

一次迸发的第一帧算作一帧 60 Hz 的帧，超过 50ms 的间隔（标签页从后台回来）算作 50ms，所以什么都不会跳。演示还把超过 1.5 帧的一步（比如在 30 Hz 屏幕上，或掉帧之后）分成两半走，这样弧线和 60 Hz 屏幕上画的保持接近。

因为循环读取实时的 `pCount`、`spreadDeg` 和 `gravity` 值，改一个设置会改变下一次迸发。再点一次只是再调用一次 `fire()`——重播是隐含的。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 彩带数量 | 中等 | 飞出多少块碎片：一些是 60，中等 120，多是 200；超过大约 200，在较慢的手机上耗费帧，收益却很小 |
| 扩散范围 | 中等 | 扇形有多宽：窄是 60°，中等 110°，宽版是 170°；窄像喷泉，宽像穹顶 |
| 下落速度 | 普通 | 碎片被往下拉得多厉害：轻飘、普通和沉重让它们的下落每秒加速大约 540、1,000 和 1,600px，而发射速度是每秒 360 到 780px，所以最快的碎片在落下前大约爬升 560、300 或 180px；沉重落得快，轻飘漂浮 |

## 生产注意事项
- **按时间，不按帧**：把每一帧的量（重力、旋转、淡出）按距离上一帧的时间缩放，阻力系数也要升到同样的幂（`0.99 ** k`）。没有它，迸发在 120 Hz 屏幕上快一倍，在 30 Hz 手机上慢一半，围绕它计时的东西，比如演示里的“演示一下”，都会错位。
- **屏幕外剔除**：一旦 `p.y` 经过舞台底部就滤掉碎片，不只是淡出时——否则快速下落的彩带会在折叠线下面继续消耗 CPU。
- **一个 rAF，带门控**：第一次迸发时启动循环，数组空了就停掉。下一次点击时重新启动。永远不要为一个偶尔发生的事件留一个永久运行的 rAF。
- **DPR 感知的 canvas**：按 `devicePixelRatio`（上限约 2）缩放后备存储，这样矩形在视网膜屏上有干净的边缘，又不会在 3x 显示器上把填充成本翻四倍。
- **矩形最便宜**：`translate`/`rotate` 里的 `fillRect`，比每个粒子画图片或路径都快。如果需要形状，一次性把精灵预渲染到离屏 canvas 上，再 `drawImage`。
- **减弱动态效果**：完全跳过彩带，显示静态的成功状态——对勾和“已下单”标签已经确认了完成。
- **canvas-confetti**：首选库——`confetti({ particleCount: 120, spread: 110, origin: {x, y} })` 处理物理、DPR 和清理。只有需要自定义形状或严格控制包大小时，才手写代码。
- **GSAP / Framer Motion**：两者都能为轻量庆祝动画少量 DOM 彩带，但对几百块碎片来说，一个 canvas 比那么多动画节点便宜得多。

## 另见
- [对勾绘制](../checkmark-draw/) — 更安静的成功表示方式
- [按钮按压缩放](../button-press-scale/) — 按下时按钮缩小
- [点赞爱心迸发](../heart-burst/) — 每次点赞的小迸发
- [弹窗展开](../modal-expand/) — 对话框从打开它的按钮处长出来
- [烟花](../../07-ambient-background/fireworks/) — 作为背景不停的火花迸发
