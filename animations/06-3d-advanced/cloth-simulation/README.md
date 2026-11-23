# 布料模拟

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/cloth-simulation/)

## 它是什么
布料模拟把布看作由固定长度的连线连起来的点网格。每秒六十次，每个点按自己原来的运动方向继续移动，再加上重力和风，然后连线把相邻的点拉回原形。固定几个点，网格就会像布一样悬挂、摇摆和折叠，还可以抓着玩。

## 什么时候用
- 主视觉区的动态旗帜和横幅
- 服装、纺织品或实物商品的产品页
- 用户可以拉扯和摆弄一块布的互动创意体验
- 浏览器里物理模拟的教学演示

## 工作原理
**Verlet 积分**——每个顶点用自己的历史更新（`dt` 是固定步长，1/60 秒）：

```js
function integrate(dt) {
  vertices.forEach((v, i) => {
    if (pinned[i]) return;

    const vx = v.x - prev[i].x;  // implicit velocity (current - previous)
    const vy = v.y - prev[i].y;

    prev[i] = { x: v.x, y: v.y }; // save current as "previous"

    v.x += vx + WIND_X * dt * dt;
    v.y += vy + GRAVITY * dt * dt;
  });
}
```

**距离约束**——积分之后，反复把相连的顶点拉回它们的静止距离：

```js
function solveConstraints(iterations) {
  for (let iter = 0; iter < iterations; iter++) {
    constraints.forEach(([a, b, restLength]) => {
      const dx = vertices[b].x - vertices[a].x;
      const dy = vertices[b].y - vertices[a].y;
      const dist = Math.sqrt(dx*dx + dy*dy) || 0.001;
      const diff = (dist - restLength) / dist * 0.5;

      if (!pinned[a]) { vertices[a].x += dx * diff; vertices[a].y += dy * diff; }
      if (!pinned[b]) { vertices[b].x -= dx * diff; vertices[b].y -= dy * diff; }
    });
  }
}
```

**固定步长**——Verlet 把一个点的速度记为它上一步移动的距离，所以每步必须等长。按自上一帧以来的时间拉长一步会改变物理结果，长的一步会把点甩到连线拉不回来的地方。演示因此始终以 1/60 秒为步长，按自上一帧以来的时间跑相应步数：

```js
const STEP = 1000 / 60;                  // one physics step, in milliseconds
let behind = 0;                          // time that has passed but not been stepped yet

function frame(elapsed) {                // elapsed: ms since the last drawn frame
  if (elapsed < 16) return;              // draw at most once per 16 ms; the time waits for a later frame
  behind += Math.min(elapsed, 50);       // a long gap counts for at most 50 ms
  for (let n = 0; behind >= STEP - 1 && n < 3; n++) {   // 1 ms of slack, at most 3 steps
    integrate(STEP / 1000);              // dt in seconds: 1/60
    solveConstraints(iterations);
    behind -= STEP;
  }
  draw();
}
```

60 Hz 的屏幕每帧跑一步，30 Hz 的跑两步。演示最多每 16ms 画一次，所以 120 Hz 的屏幕每隔一次刷新画一帧、跑一步，144 Hz 的屏幕每隔三次刷新（每秒约 48 张图）画一帧、跑一到两步。布在每种屏幕上落下和摆动的速度都一样。1ms 的宽限让来得稍早一丝的帧不会被跳过、然后又补算两次，最多三步的上限意味着跟不上的设备只会让布变慢，而不是堆积更多工作。

**渲染**——三角化的网格画成填充的四边形。每个四边形沿对角线分成两个三角形：

```js
for (let r = 0; r < ROWS; r++) {
  for (let c = 0; c < COLS; c++) {
    const a = vertices[idx(r,   c  )];  const b = vertices[idx(r,   c+1)];
    const d = vertices[idx(r+1, c  )];  const e = vertices[idx(r+1, c+1)];
    ctx.beginPath(); ctx.moveTo(a.x,a.y); ctx.lineTo(b.x,b.y); ctx.lineTo(d.x,d.y); ctx.fill();
    ctx.beginPath(); ctx.moveTo(b.x,b.y); ctx.lineTo(e.x,e.y); ctx.lineTo(d.x,d.y); ctx.fill();
  }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 挂在 | 两个角 | 固定顶部两个角、整条顶边、一个角，或什么都不固定，那样布会掉到地上 |
| 风 | 中等 | 风吹得多大；风力会在 12.6 秒的周期里自行起伏：峰值时轻风每秒每秒让点横向加速约 30px，中等 75px，强烈 180px，有一段时间还会以最多约一半的力度反向吹 |
| 硬度 | 中等 | 每步（每秒六十步）把连线拉回原长的次数：有弹性是 2 次，中等 4 次，硬挺 8 次 |
| 重力 | 普通 | 布有多重：重力让下落的点每秒每秒加速，轻约 17px，普通 35px，重 58px |
| 细节 | 中等 | 布的网格有多少格子，每个角上一个点：低是 16 × 12，中等 24 × 18，高是 40 × 30；布裁成舞台宽的 80%、高的 65%，所以格子的形状跟着舞台走：手机上大约是正方形，笔记本上宽是高的两到三倍；手机从低开始 |
| 显示点 | 关 | 标出网格上的每个点，固定住的点是红色的 |

## 生产注意事项
- **Verlet 与显式欧拉**：显式欧拉积分把速度显式存下来，每步加到位置上。Verlet 积分对受约束的系统数值上更稳定——大步长下布的弹簧不那么容易“炸开”。
- **固定时间步长**：每绘制一帧走一步，布在 120 Hz 的屏幕上会落两倍快，在 30 Hz 的手机上慢一半，而把一步拉长到一帧的长度会改变 Verlet 的计算结果。像上面那样从累加器里按固定 1/60 秒走步，每帧画一次。
- **弯曲约束**：要做出真实的布，加上剪切约束（对角邻居）和弯曲约束（隔一个顶点）。演示为简单起见只用结构约束（直接邻居）。
- **GPU 布料模拟**：3A 游戏用计算着色器（DirectX/Vulkan/Metal）或 GPGPU 通道（与 GPGPU 粒子系统演示相同的乒乓纹理技术）在 GPU 上模拟布料。Chrome 113+ 起浏览器里可用 WebGPU 实现。
- **Cannon.js / Rapier.js**：包含布料/软体模拟的 JavaScript 物理引擎，精度更高、约束类型更多。Rapier（Rust/WASM）是目前最快的选择。
- **自碰撞**：演示不阻止布穿过自身（自相交）。自碰撞检测是 O(n²)，需要空间哈希或 BVH 加速结构。
- **性能**：40×30（1271 个点、2470 条连线，每步 4 遍 = 9880 次连线修正）时，手机级 CPU 每秒约 17 帧（每帧一步一画），24×18 时约 47 帧。演示在手机（宽 600px 及以下，或高 500px 及以下）上从 16×12 开始，那里能保持 60 帧；「细节」设置仍然可以选更多。
- **画布分辨率**：后备存储按 `devicePixelRatio`（封顶 2）设置，上下文相应缩放，这样网格在手机上依然清晰，又不用付出 3 倍帧缓冲的代价。

## 另见
- [GPGPU 粒子系统](../gpgpu-particle-system/) — 移到显卡上的物理
- [噪声驱动运动](../noise-based-motion/) — 用噪声代替物理的自然运动
- [Canvas 粒子效果](../canvas-particle-effect/) — 给松散粒子用的更简单的物理
- [2D 物理](../physics-2d/) — 落下、弹跳、堆积的实心球
