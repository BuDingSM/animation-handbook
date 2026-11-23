# 2D 物理

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/physics-2d/)

## 它是什么
2D 物理按小步模拟时间，让物体像真实物体那样落下、弹跳和落定。每步给每个球的速度加上重力，找出互相接触或碰到墙的球，再把它们推开刚好不陷进去的距离，保留一部分速度作为弹跳。每秒六十步这样的几条规则，就足够让球弹跳、滚到停下、堆起来，还能被拿起和扔出去。

## 什么时候用
- 对每次点击或轻触都有反应的俏皮主视觉区和开场
- 空状态、404 页面和等待画面，给用户一点可以玩的东西
- 游戏、玩具和产品页，人们在那里扔、堆或推倒东西
- 讲重力、弹跳和摩擦的教学页面

## 工作原理
**固定步长。**物理步长必须始终等长，否则弹跳和堆积在每个屏幕上的表现都不一样。循环把还没走完的时间留在 `behind` 里，按它能走的步数跑 1/60 秒的步，和布料模拟的做法一样；慢动作只加三分之一的时间：

```js
const FRAME = 1000 / 60;                       // one step, in ms
function loop(ts) {
  raf = requestAnimationFrame(loop);
  if (lastT === null) { lastT = ts; return; }  // the first frame after a start only takes the time
  const dt = ts - lastT; if (dt < 16) return;  // draw at most once per 16 ms
  lastT = ts;
  const ms = Math.min(dt, 50) * (slow ? 1 / 3 : 1);
  if (awake) {
    behind += ms;
    for (let n = 0; behind >= FRAME - 1 && n < 3; n++) { step(); behind -= FRAME; }
  }
  fade(ms); draw();
  if (!awake && nothingFades()) { cancelAnimationFrame(raf); raf = null; }   // still: stop drawing
}
```

**接触。**每步先加上重力，再列出每个接触的球对和每个碰到墙的球。一个接触记录它的方向 `n`，如果两者靠近的速度快于每步 1px，记录它们应该分开的速度：弹性乘以靠近的速度。更慢的接触没有弹跳，所以静止的球会保持静止而不是颤抖：

```js
const touch = (a, b, nx, ny) => {               // n points from a to b; a wall is a body that never moves
  const vn = (b.vx - a.vx) * nx + (b.vy - a.vy) * ny;   // negative: they are closing
  contacts.push({ a, b, nx, ny, t: vn < -1 ? -E * vn : 0, j: 0, jt: 0 });
};
```

**通过堆积传递推力。**对所有接触做八遍，每遍加上让一对球达到分开速度的推力，累计总数永远不会变成拉力。堆深处的球同时被几个邻居推，几遍下来推力在它们之间落定。摩擦沿接触面以同样的方式工作，限制在推力的一半，这正是球能堆起来而不是滑开的原因：

```js
for (let pass = 0; pass < 8; pass++) for (const c of contacts) {
  const { a, b, nx, ny } = c, ia = a.im, ib = b.im, s = ia + ib;      // im: 1 / mass (0 for a wall or a held ball)
  const j = Math.max(0, c.j + (c.t - ((b.vx - a.vx) * nx + (b.vy - a.vy) * ny)) / s), dj = j - c.j; c.j = j;
  const f = 0.5 * c.j, jt = clamp(c.jt - ((b.vx - a.vx) * -ny + (b.vy - a.vy) * nx) / s, -f, f), dt = jt - c.jt; c.jt = jt;
  const px = dj * nx - dt * ny, py = dj * ny + dt * nx;                  // the push plus the friction
  a.vx -= px * ia; a.vy -= py * ia; b.vx += px * ib; b.vy += py * ib;
}
```

然后每个球按速度移动（每步不超过约一个半径，这样快球不会穿过另一个），三遍把剩下的重叠缓缓分开，每个球都留在盒子里。球的质量随面积增长，所以大球会把小球推到一边。

**休眠。**45 步（四分之三秒）里没有球移动得比每步约 0.14px 更快时，所有速度设为零，循环停止绘制。新球、一次投掷、一个被移走的球或新的重力会再次唤醒它。

**投掷。**按下会拿起够得着的最近的球，够得着的范围是它的半径加 8px，绝不小于一个 44px 的圆，所以小球用手指也容易抓住。拿着的球跟随指针，什么也不推它，而它会推别的球。指针移动给出指针每六十分之一秒的像素速度，经过最近几次移动平滑；松手时，如果指针在最近 80ms 里还在动，球就以那个速度被扔出去。

**减弱动态。**屏幕上什么也不落下。一次落下或投掷的步数会一下子在看不见的地方跑完，直到一切静止，新球在它落定的地方淡入：

```js
function settle() { awake = true; calm = 0; for (let i = 0; i < 900 && awake; i++) step(); awake = false; }
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 重力 | 地球 | 球被往下拽的力度。地球每六十分之一秒给球的向下速度加上盒子高度除以 1000，球约四分之三秒落完整盒子；月球的拉力是五分之一，约 1.6 秒落下，重是两倍，约半秒 |
| 弹性 | 中等 | 球撞击后保留的靠近速度占比：低是 0.2，中等 0.5，高是 0.8；慢于每六十分之一秒 1 像素的撞击不弹跳 |
| 球的大小 | 全部相同 | 每个球一个半径（盒子较短边的 5.5%，在 12 到 22 像素之间），或 0.6 到 1.4 倍的混合，按面积更重 |
| 显示速度 | 关 | 每个运动的球上一个箭头，每六十分之一秒移动 1 像素对应 6 像素长，最多三个半径 |

## 生产注意事项
- **库**：Matter.js 在 2D 里处理圆形、方块、多边形、关节和休眠；Planck.js 是 Box2D 的移植版，是许多游戏背后的引擎；Rapier（Rust 编译成 WebAssembly）在更高的物体数量下做 2D 和 3D。一旦需要方块、多边形或铰链就用它们。GSAP 和 Framer Motion 朝设定值做动画，它们的弹簧和弹跳缓动只模拟一次落地，不是堆积。
- **只有圆形**：演示没有转动的物体；每个球上的斑点随横向速度转动，读起来像滚动。方块和多边形需要角速度、接触点和多边形碰撞检测。
- **固定步长**：按帧时间缩放一步会让慢屏幕上的弹跳更高、堆积抖动。从累加器里按固定 1/60 秒走步，封顶每帧的步数（这里是 3 步），慢设备只会让物理变慢而不是落后。
- **隧穿**：一步移动超过自身大小的球可能跳过另一个。演示把速度封顶在每步约一个半径；引擎用连续碰撞检测或更小的步长。
- **休眠**在一切静止后停掉工作，节省手机电量；任何输入都能唤醒物体。
- **每对都会检查**：演示保留的 40 个球没问题（手机上 24 个，宽 600px 及以下或高 500px 及以下的屏幕）；几百个时先把物体排进网格。
- **触摸**：画布关掉浏览器的触摸滚动（`touch-action: none`），所以拖动移动的是球而不是页面；页面仍然能从盒子外面滚动。

## 另见
- [布料模拟](../cloth-simulation/) — 固定步长的物理，点与点之间由连线连接
- [弹跳进入](../../02-entrance-and-exit/bounce-in/) — 一次弹簧式落地，用关键帧而不是物理做出
- [鸟群](../flocking/) — 另一个模拟，几条简单规则做出栩栩如生的运动
