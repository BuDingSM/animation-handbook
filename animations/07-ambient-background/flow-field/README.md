# 流场

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/flow-field/)

## 它是什么
流场让成千上万的粒子穿过画布，每个粒子按它下方看不见的水流方向转向。方向从一处到另一处平滑变化，所以彼此靠近的粒子一起弯曲，整个表面呈现出轻柔、河流般的水流。画布每帧只是变暗一点而不是清空，所以每个粒子都留下一条渐隐的轨迹，描出水流。

## 什么时候用
- 生成式、有机的主视觉背景，每次加载看起来都略有不同
- 数据艺术和创意编程作品
- 深色落地页后面的氛围循环，想要质感而不是主体
- 过渡或加载画面，持续运动对它有好处

## 工作原理
没有存储的场——角度是按需从一个内联值噪点函数（哈希格子，平滑插值）算出的。每个粒子采样它下方的角度，朝那个方向走一步，离开画布时重生。轨迹来自画半透明背景而不是清空：

```js
function vnoise(x, y) {
  const xi = Math.floor(x), yi = Math.floor(y), xf = x - xi, yf = y - yi;
  const tl = hash(xi,yi), tr = hash(xi+1,yi), bl = hash(xi,yi+1), br = hash(xi+1,yi+1);
  const u = smooth(xf), v = smooth(yf);
  return (tl*(1-u)+tr*u)*(1-v) + (bl*(1-u)+br*u)*v;
}
function angleAt(x, y, time) {
  return vnoise(x/SCALE, y/SCALE + time*0.15) * Math.PI * 4;
}

// per drawn frame: fade, then advance every particle along its local angle
// s is how many 60 Hz frames have passed since the last drawn frame: 1 at 60 Hz, 2 at 30 Hz
const s = Math.min(now - last, 50) / (1000 / 60), w = Math.max(1, Math.round(s));
const f = fa + (Math.abs(s - w) < 0.05 ? w : s);   // a step within 5% of a whole number counts as that number
const nf = Math.round(f + 0.05);                    // whole dims due; the 0.05 keeps a half-way tie off the rounding edge
if (!nf) return;                                    // none yet: draw nothing, the time carries to the next frame
fa = f - nf; last = now;                            // the fraction left over carries too
ctx.fillStyle = `rgba(5,6,10,${TRAIL})`;
for (let i = 0; i < nf; i++) ctx.fillRect(0, 0, W, H);
const k = s * (slow ? 1 / 3 : 1);                   // slow motion shortens the step, not the dim
const a = angleAt(p.x, p.y, t);
p.x += Math.cos(a) * SPD * k; p.y += Math.sin(a) * SPD * k;
```

步长随自上一帧绘制以来的时间增长，淡出也是，以整个单位计：画布每六十分之一秒变暗一次，每次都用与 60 Hz 时相同的 `TRAIL`，所以 30 Hz 的一帧变暗两次。离整数 5% 以内的步长在变暗时算作那个整数，所以 59.94 或 60.06 Hz 的屏幕，或时间戳里的一点噪点，每帧仍然恰好变暗一次，`fa` 带着剩下的小数部分。四舍五入向上偏 0.05，所以帧正好落在两次变暗中间的屏幕，比如 120 或 240 Hz，能保持规律的绘制节奏，而不是由时间戳噪点决定。还没有整次变暗到期时，比如 120 Hz 屏幕两帧中的第一帧，那一帧什么也不画，它的时间带到下一帧。这样无论什么屏幕，绘制最多大约每秒 60 帧（30 Hz 的屏幕画 30 帧，每帧变暗两次），每帧绘制都变暗一次，所以轨迹不会脉动。30 Hz 的手机和 144 Hz 的显示器以相同速度显示粒子，轨迹以秒计的长度也相同。一次 `1 - (1 - TRAIL)^s` 的变暗理论上淡出的量相同，但 8 位画布每次变暗都会四舍五入，所以淡淡的残留会在不同屏幕上沉淀成不同颜色；重复同样的变暗让它们在所有地方沉淀成相同的颜色。开始、暂停或从隐藏标签页返回后的第一帧什么也不加，长间隔最多算 50ms。到达时画的静止画面运行 40 步 `s = 1`，每步变暗一次，所以在每个屏幕上看起来都一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 漩涡大小 | 中等 | 水流有多大：方向大约在 20px（小）、34px（中等）或 60px（大）内转一圈；越大越平静，越小越湍急 |
| 轨迹长度 | 中等 | 轨迹留存多久：短是每六十分之一秒淡 12%，中等 6%，长是 4%；低于约 4% 时轨迹会留下永不淡去的淡淡痕迹 |
| 速度 | 普通 | 每个粒子在六十分之一秒内移动多远：慢速 0.6px，普通 1px，快速 1.6px（每秒 36、60 和 96px） |
| 粒子数量 | 中等 | 少是 400 个，中等 900 个，多是 1500 个；手机上最多显示 500 个 |
| 颜色 | 薄荷绿 | 薄荷绿、余烬或冰色，或彩虹色，颜色在舞台上和随时间变化 |

## 生产注意事项
- **轨迹透明度与残留**：因为轨迹依赖不完全清空，非常低的留存值会在某些 GPU 上留下永久残留。如果看到永不完全淡去的重影，就把它调高一点（0.04 以上）。
- **手机上限**：手机尺寸的屏幕（宽最多 600px，或横持手机时高最多 500px）上粒子数量限制为 500 个。每个粒子是一段描画的线段，所以手机上的瓶颈是填充率——不是数学计算。
- **确定性噪点**：`hash` 用的是 `sin(x*127.1 + y*311.7)*43758.5453`，经典的 GLSL 技巧。它不是加密用的，也不是真正的柏林噪点，但便宜、无依赖，对一个场来说够平滑了。
- **减少动态**：演示开始时是暂停的，显示约 40 帧一次画成的静止、稳定的画面，直到访客按下播放。每次打开页面都画同样的 40 帧，所以舞台永远不会从空开始。
- **库的等价实现**：生产环境的流场通常跑在 GPU 上——[three.js](https://threejs.org) 加片段/计算着色器，或粒子系统里的旋度噪点。[tsParticles](https://github.com/tsparticles/tsparticles) 不做真正的流场，但它的路径插件近似有方向的运动。

## 另见
- [粒子星座](../particle-constellation/) — 连起来而不是流动的粒子
- [极光](../aurora/) — 用模糊做出的流动彩色光带
- [网格渐变](../mesh-gradient/) — 没有粒子的平滑漂移色彩
- [等高线](../topographic-lines/) — 同种噪点，画成地图线而不是轨迹
