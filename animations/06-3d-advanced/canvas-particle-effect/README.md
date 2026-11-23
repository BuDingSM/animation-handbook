# Canvas 粒子效果

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/canvas-particle-effect/)

## 它是什么
Canvas 粒子效果在一块画布上画出几百个小圆点，每帧移动它们。每个圆点朝自己的方向漂移，碰到边缘就弹回来；靠得近的圆点之间会连上细线，指针会把附近的圆点推开或拉近。这就是许多科技网站上的连线圆点背景。

## 什么时候用
- 需要动感又不抢前景内容的主视觉区背景
- 网络或数据可视化的隐喻（节点代表连接）
- 深色主题仪表盘上的氛围装饰
- 任何以“技术 / 互联系统”为视觉语言的场景

## 工作原理
每个粒子存储位置、速度、半径和不透明度。每帧，位置按速度更新，在作用半径内时鼠标的力会推动速度。两者都乘以 `k`，即自上一帧以来的六十分之一秒步数（60 Hz 的屏幕上是 1，30 Hz 上是 2，慢动作里是它的三分之一），所以圆点在任何屏幕上都以同样的速度漂移：

```js
class Particle {
  update(k) {
    // Mouse repel/attract
    const dx = this.x - mouse.x, dy = this.y - mouse.y;
    const d2 = dx * dx + dy * dy;
    if (d2 < 14400 && d2 > 1) {         // within 120px
      const d = Math.sqrt(d2);
      // (dx, dy) / d points away from the cursor, so subtracting FORCE
      // makes a negative value repel and a positive value attract.
      this.vx -= FORCE * (dx / d) / d * 0.8 * k;
      this.vy -= FORCE * (dy / d) / d * 0.8 * k;
    }
    // Speed cap
    const sp = Math.sqrt(this.vx**2 + this.vy**2);
    if (sp > MAX_SPEED) { this.vx = this.vx/sp * MAX_SPEED; this.vy = this.vy/sp * MAX_SPEED; }
    // Bounce off walls
    this.x += this.vx * k; this.y += this.vy * k;
    if (this.x < 0 || this.x > W) this.vx *= -1;
    if (this.y < 0 || this.y > H) this.vy *= -1;
  }
}

let owed = 0;                 // steps of Trails wash not laid yet: 0 to start with, and cleared again when the loop restarts (Play, a tab coming back)

function frame(elapsed) {     // called for every frame the screen offers; elapsed: ms since the last frame that was drawn
  const steps = Math.min(elapsed, 50) / (1000 / 60);   // a long gap counts for at most 50 ms
  const whole = Math.max(1, Math.round(steps));
  const due = Math.abs(steps - whole) < 0.05 ? whole : steps;   // within 0.05 of a whole number counts as that number
  if (Math.round(owed + due + 0.05) < 1) return;   // no wash due: draw nothing and change nothing, the time waits for the next frame

  const k = steps * (slow ? 1 / 3 : 1);
  particles.forEach(p => p.update(k));      // Slow motion slows the dots ...
  owed += due;                              // ... but not the Trails fade: one wash of 15% for every step
  const washes = Math.round(owed + 0.05);   // (Math.round, so a frame a little short of 1/60 s still washes once)
  owed -= washes;
  draw(washes);                             // lays the wash over the picture that many times, then draws the dots
}
```

覆盖次数是单独算出来的，这样略微偏离 60 Hz 的屏幕（59.94 或 60.06）或帧到达时间略不均匀时，每帧仍然恰好覆盖一次。舍入前加的 0.05，让恰好 120 和 240 Hz 的屏幕上卡在绘制决策处的那半步累计时间，不会落在时间戳噪点可能两边倒的边界上。覆盖还没到期的那一帧什么也不画（「拖影」关掉时也一样，因为同样的计数决定节奏），这也让更快的屏幕上绘制保持在每秒 60 张图附近。

**连线**——每帧 O(n²) 的距离检查：

```js
// Every link goes into one of six paths by its opacity; each path is stroked once
const paths = Array.from({ length: 6 }, () => new Path2D());
for (let i = 0; i < particles.length; i++) {
  for (let j = i + 1; j < particles.length; j++) {
    const dx = particles[i].x - particles[j].x;
    const dy = particles[i].y - particles[j].y;
    const d2 = dx*dx + dy*dy;
    if (d2 < DIST * DIST) {
      const k = Math.min(5, Math.floor((1 - d2 / (DIST * DIST)) * 6));
      paths[k].moveTo(particles[i].x, particles[i].y);
      paths[k].lineTo(particles[j].x, particles[j].y);
    }
  }
}
ctx.lineWidth = 0.5;
paths.forEach((path, k) => {
  ctx.strokeStyle = `rgba(88, 166, 255, ${(k + 0.5) / 6 * 0.4})`;
  ctx.stroke(path);
});
```

每条连线单独描画，每条线要一次绘制调用，一帧要几千次；把连线按六档不透明度分组，六次调用就全画完了，肉眼看不出差别。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 指针效果 | 推开 | 指针或手指对 120px 内圆点的作用：推开、拉近，或什么都不做，越近作用越强（40px 外的圆点每秒每秒加速约 36px），最多到最快漂移速度的两倍；圆点会保留它获得的速度 |
| 粒子数量 | 中等 | 电脑上少是 250 个，中等 500 个，多是 800 个圆点；平板画 60、90 和 120 个，手机画 30、45 和 60 个，这样能保持流畅 |
| 连线距离 | 中等 | 两个圆点要多近才会连上线：短是 70px，中等 100px，长是 150px；无不画线 |
| 速度 | 普通 | 圆点漂移的速度：最快的圆点慢是每秒 30px，普通 48px，快 78px，最慢的只有它的五分之一，在任何屏幕上都一样；慢显得安静，快显得热闹 |
| 颜色 | 蓝色 | 圆点和连线的颜色 |
| 拖影 | 关 | 每个圆点会留下一道渐渐消失的拖影，因为画布在帧之间只被部分清除 |

## 生产注意事项
- **O(n²) 上限**：所有粒子对之间的距离检查按平方增长。超过约 500 个粒子，循环就会掉帧。解决办法：空间划分（四叉树、均匀网格）把检查降到 O(n log n)。1000 个粒子以上改用 WebGL。
- **手机和平板**：演示在手机（宽 600px 及以下，或横持时高 500px 及以下）上画 30 到 60 个圆点，在宽 1024px 及以下的其他屏幕上画 60 到 120 个，连线分六批描画，而不是每条线一次调用。
- **Canvas 与 DOM**：50 个粒子以上必须用 `<canvas>`。这个密度的 DOM 元素每帧会产生几千次布局计算，浏览器跟不上。
- **Particles.js / tsParticles**：生产环境的主流库。除了演示里的全部功能，还有主题、形状多样性、响应式密度和高数量下的性能。
- **`ctx.clearRect` 与 `fillRect`**：用半透明背景的 `fillRect` 代替 `clearRect`，会产生旧帧 lingering 的拖影效果（在演示里打开「拖影」）。演示每六十分之一秒的帧时间覆盖一次（30 Hz 的屏幕上两次，59.94 Hz 以上的每个绘制帧一次），始终是 15%，与「慢动作」无关，所以拖影在 30 Hz 的手机上和 60 Hz 的屏幕上持续一样久。一次更小的覆盖淡出的量理论上相同，但 8 位画布每次覆盖都会四舍五入，拖影淡淡的残留最后会变成另一个颜色。
- **按时间，不按帧**：每帧固定一步会跟着屏幕的刷新率走，圆点在 120 Hz 下漂两倍快，30 Hz 下慢一半。封顶一帧的时间（演示用 50ms），让回来的标签页不至于猛地一跳，并让回来的标签页从下一帧开始。
- **设备像素比**：后备存储按 `clientWidth * devicePixelRatio`（封顶 2）设置，并缩放上下文，否则视网膜屏上的亚像素圆点和 0.5px 连线会发虚。
- **`prefers-reduced-motion`**：停掉所有粒子运动。可以把静态的圆点布局留下来当作纹理。

## 另见
- [GPGPU 粒子系统](../gpgpu-particle-system/) — 在显卡上移动的数万个粒子
- [噪声驱动运动](../noise-based-motion/) — 由平滑噪声移动的圆点，而不是物理
- [粒子星座](../../07-ambient-background/particle-constellation/) — 更安静的连线圆点，作为背景
- [鸟群](../flocking/) — 跟随邻居、成群盘旋的圆点
- [文字粒子](../../05-text-typography/text-particles/) — 拼成词的圆点从指针处散开再弹回来
