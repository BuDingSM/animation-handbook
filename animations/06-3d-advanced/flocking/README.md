# 鸟群

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/flocking/)

## 它是什么
鸟群让一群简单的圆点像一群鸟或一群鱼一样整体移动，没有领头，也没有计划。每只鸟只看得见附近的鸟，遵循三条规则：与最近的鸟保持一点距离、按邻居的方向飞、向群体的中间靠拢。这三条局部规则就足以让整个鸟群盘旋、分开再汇合；Craig Reynolds 在 1986 年描述它们时，把这些鸟叫做“boids”。

## 什么时候用
- 主视觉标题后面安静、有生命力的背景
- 自然、旅行和户外品牌：鸟、鱼、昆虫、无人机
- 游戏、艺术页面和生成式设计展示
- 讲解蜂群、人群或涌现行为的文章

## 工作原理
每一步，每只鸟收集视野半径 `R` 内、位于它前方或侧面的鸟（一只鸟正后方有个盲区），再按最多三股转向拉力转向：

```js
function step(k) {                       // k: the length of this move in 60 Hz frames, at most 1.05
  for (const b of birds) {
    let n = 0, cx = 0, cy = 0, ax = 0, ay = 0, sx = 0, sy = 0;
    const vb = Math.hypot(b.vx, b.vy);
    for (const o of birds) {
      if (o === b) continue;
      const dx = o.x - b.x, dy = o.y - b.y, d2 = dx * dx + dy * dy;
      if (d2 > R * R || dx * b.vx + dy * b.vy < -0.5 * vb * Math.sqrt(d2)) continue; // too far, or behind
      n++; cx += o.x; cy += o.y;                                // stay close: their middle
      ax += o.vx; ay += o.vy;                                   // match direction: their heading
      if (d2 < SEP * SEP) { sx -= dx / d2; sy -= dy / d2; }     // keep apart: a closer one pushes harder
    }
    acc.x = acc.y = 0;
    if (n) {
      steer(b, ax, ay, MAX_TURN, ALIGN);                 // turn toward the neighbors' heading
      steer(b, cx / n - b.x, cy / n - b.y, MAX_TURN, COHERE); // turn toward their middle
    }
    steer(b, sx, sy, MAX_TURN * 1.5, 2);                 // turn away from the closest ones, harder than the rest
    b.vx += acc.x * k; b.vy += acc.y * k;                // the turn, scaled by the time that passed
    keepSpeedBetween(b, 0.55 * MAX_SPEED, MAX_SPEED);    // birds never hover or race
    b.x += b.vx * k; b.y += b.vy * k;
  }
}
```

每股拉力都是 Reynolds 转向：让鸟以全速指向想要的方向所需的速度变化，封顶后一只鸟一步只能转这么急：

```js
function steer(b, dx, dy, cap, weight) {
  const d = Math.hypot(dx, dy); if (!d) return;
  let x = dx / d * MAX_SPEED - b.vx, y = dy / d * MAX_SPEED - b.vy;
  const l = Math.hypot(x, y);
  if (l > cap) { x *= cap / l; y *= cap / l; }
  acc.x += x * weight; acc.y += y * weight;
}
```

演示还加了两股拉力。沿每条边缘的留白里，鸟会往回转，越深入留白转得越急，所以鸟群会从边缘处弯回来，而不是堆在边缘上。打开「躲开指针」后，指针或手指附近的鸟会以四倍于平常的转向直直躲开。

**按时间，不按帧。**循环最多每 16ms 画一次，按自上次绘制以来的时间（以 60 Hz 帧计，封顶 50ms）让鸟飞，所以它们在 30、60 或 120 Hz 的屏幕上飞得一样快；慢动作只算它的三分之一。比一帧 60 Hz 的帧长 5% 以上的一帧，会飞成几步等长的移动，每步不超过 1.05 个 60 Hz 帧（这 5% 让来得稍晚一丝的帧仍算一步）。于是鸟一步转的弯永远不会比 60 Hz 时急多少，30 Hz 的屏幕以恰好一帧的两步，飞出与 60 Hz 完全相同的路径：

```js
function advance(k) { const n = Math.max(1, Math.ceil(k - 0.05)); for (let i = 0; i < n; i++) step(k / n); }
```

**绘制。**每只鸟是沿速度方向的箭头；所有鸟放进一条路径，每帧填充一次。

**第一张图就有鸟群。**新的鸟群从随机位置出发，先飞三秒（180 步）再第一次画出来，所以第一张图、以及减弱动态下显示的静止图，呈现的已经是鸟群。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 鸟群大小 | 中等 | 飞着几只鸟：小是 70 只，中等 150 只，大是 280 只；手机上飞 35、70 和 130 只 |
| 飞得多近 | 普通 | 每只鸟在身边留出的空间，以及它跟随邻居的力度：松散留 32px、跟随得轻柔，普通留 22px，紧密留 15px、跟随得用力。朝向邻居方向和朝向它们中间的拉力，松散时权重是 0.7 和 0.3，普通时 1 和 0.5，紧密时 1.2 和 0.9，而保持距离的权重是 2；在高或宽不到 380px 的舞台上，空间按比例缩小，最少到 65% |
| 躲开指针 | 开 | 指针或手指 90px 内的鸟会直直躲开 |
| 速度 | 普通 | 最高速度，以每六十分之一秒的像素计：慢是 1.3，普通 2，快是 3（每秒 78、120 和 180px）；没有鸟飞得比它慢 55%。在高或宽不到 380px 的舞台上按比例缩小，最少到 65% |
| 显示一只鸟的视野 | 关 | 用 50px 半径的圆圈出一只鸟和它保持的虚线空间，并连上它跟随的每个邻居 |

在手机这样的小舞台上，所有距离和速度最多缩小 35%。

## 生产注意事项
- **每只鸟都要检查其他每只鸟**，所以工作量随鸟群的平方增长：280 只鸟一步约 78000 次检查，仍在 2ms 以内。几千只鸟时，把它们放进与视野半径等大的格子，只检查每只鸟周围九个格子，或者把鸟群移到显卡上（three.js 有个著名的 GPU 鸟群示例）。
- **边缘**：把鸟绕到对面边缘更简单，但鸟群会在边缘处断开。在留白里往回转能让每个鸟群都留在视野里。
- **速度下限**（这里是最高速度的 55%）让鸟不会悬停在原地，那样看起来不像鸟；鱼可以飞得更慢。
- **权重决定观感**：朝中间和朝向的拉力越大，椋鸟群越紧密、越密集；空间越大、拉力越小，群体越松散、越漂移。
- **库**：p5.js 和 Processing 自带经典的鸟群示例；游戏引擎和 three.js 能在 3D 里做。GSAP 和 Framer Motion 动画的是固定路径，不是智能体，所以鸟群要手写或用模拟库。
- **手机**飞的鸟大约只有一半，这样帧时间低，小舞台也更清晰。
- **减弱动态**：让鸟群静止。因为演示在第一张图之前先飞了三秒，静止帧呈现的是真正的鸟群，而不是散落的鸟。

## 另见
- [Canvas 粒子效果](../canvas-particle-effect/) — 漂移并对指针有反应的圆点，但不跟随彼此
- [流场](../../07-ambient-background/flow-field/) — 乘着看不见的水流而不是跟随邻居的粒子
- [GPGPU 粒子系统](../gpgpu-particle-system/) — 在显卡上移动的数万个粒子
- [生命游戏](../../07-ambient-background/game-of-life/) — 网格上按邻居生死的细胞
