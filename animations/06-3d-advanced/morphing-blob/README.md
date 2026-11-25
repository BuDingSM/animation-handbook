# 变形色块

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/morphing-blob/)

## 它是什么
变形色块是由普通圆圈做成的柔软液体形状。圆圈先模糊，再把边缘重新锐化，所以两个圆圈一靠近就会融成一个形状，像水银滴。随着它们漂移，色块拉伸、分裂、汇合，其中一滴跟随指针。

## 什么时候用
- 俏皮的主视觉背景和落地页焦点
- 需要感觉活着、又不分散注意力的加载和闲置状态
- 形状跟随或“抓住”指针的光标反应点缀
- 想要有机、非几何个性的 Logo 和品牌时刻

## 工作原理
每个色块都是一个普通的 SVG `<circle>`。整个组经过一个“粘液”滤镜：高斯模糊扩散每个圆圈的边缘，然后 `feColorMatrix` 把 alpha 通道推过一条陡峭的对比曲线，把柔和的模糊重新收成硬边缘。两个模糊圆圈重叠的地方，合并的 alpha 越过阈值，它们就读作一个连通的形状。

```html
<filter id="goo">
  <feGaussianBlur in="SourceGraphic" stdDeviation="10" id="blur" result="blur"/>
  <feColorMatrix in="blur" mode="matrix"
    values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 20 -9" result="goo"/>
  <feBlend in="SourceGraphic" in2="goo"/>
</filter>
<g filter="url(#goo)" fill="url(#fill)"><!-- circles --></g>
```

矩阵的最后一行（`0 0 0 20 -9`）是关键：它把 alpha 乘以 20 再减 9，于是中间范围（模糊的）alpha 塌到 0 或 1——一个硬轮廓。一个 requestAnimationFrame 循环每帧运行两个小函数。`step(dt)` 推进时间：它推进漂移相位，并让一个“水滴”圆圈缓动着靠近指针。`place()` 再算出每个圆圈的位置（背景圆圈沿着由相位驱动的正弦路径走），写进圆圈的属性：

```js
const FRAME = 1000 / 60;                  // one frame on a 60 Hz screen, in milliseconds
function step(dt){                        // dt: milliseconds since the last frame, at most 50
  phase += dt / 1000 * speed;             // integrated: dt is time, not a frame count
  const pull = 1 - Math.pow(1 - 0.09, dt / FRAME);  // 0.09 of the gap per 1/60 s, on any screen
  drop.x += (pointer.x - drop.x) * pull;  // attraction toward the cursor
  drop.y += (pointer.y - drop.y) * pull;  // (the pointer comes from Pointer Events → mouse, pen, touch)
}
function place(){
  for (const b of blobs){
    if (!b.droplet){                     // the droplet keeps its own position
      b.x = b.bx + Math.sin(phase*b.sx + b.px) * b.ax;
      b.y = b.by + Math.cos(phase*b.sy + b.py) * b.ay;
    }
    b.el.setAttribute('cx', b.x); b.el.setAttribute('cy', b.y); b.el.setAttribute('r', b.r);
  }
}
```

漂移相位是累加出来的（`phase += dt / 1000 * speed`），而不是由绝对时间算出来的。把流逝的时间乘以速度，会让每个色块在速度一改的瞬间跳到正弦路径上的一个新点；累加相位只改变速率，位置保持连续。

水滴每六十分之一秒弥合与指针之间 9% 的距离。其他长度的帧，要弥合的占比是 `1 - Math.pow(1 - 0.09, dt / FRAME)`：60 Hz 时是 0.09，30 Hz 时约 0.17，144 Hz 时约 0.04。所以水滴在任何屏幕上到达的时间都相同；每帧固定 0.09 的话，在 30 Hz 的手机上要两倍久，在 120 Hz 下只要一半。

因为漂移圆圈的位置只取决于相位，`place()` 也可以在什么都不动的情况下画出色块。演示在加载时调用一次，所以第一张图已经是色块，「播放」恰好从那里继续；暂停时改变色块数量也会再调用一次。暂停取消待处理的帧；「播放」忘记上一帧的时间，所以它之后的第一帧什么也不加，从后台回来的标签页也一样。慢动作只加三分之一的相位，让水滴每六十分之一秒弥合 0.03 的差距而不是 0.09。

指针的位置用 SVG 的屏幕矩阵（`svg.getScreenCTM().inverse()`）换算成绘图自己的单位，所以无论 SVG 怎样缩放和居中，水滴都停在指针下面，宽舞台上绘图旁边也一样。`viewBox` 是色块中间周围一个 360 单位的盒子（`20 20 360 360`），`preserveAspectRatio="xMidYMid meet"`：圆圈最多离中间约 190 单位，所以盒子几乎总是能装下它们，整个盒子适配舞台的任何形状。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 黏糊程度 | 中等 | 边缘重新锐化之前，圆圈被模糊多少：低是 6，中等 10，高是 16，以一幅 360 单位宽、圆圈半径 26 到 52 单位的绘图计量；越高，圆圈隔得越远就开始融化 |
| 色块数量 | 中等 | 少是 4 个，中等 6 个，多是 9 个圆圈，算上跟随指针的那滴 |
| 速度 | 普通 | 圆圈漂移的速度：慢是 0.6 倍，普通 1 倍，快是 1.6 倍；每个圆圈同时左右和上下摆动，每次摆动的速率各不相同（普通时一次摆动要 6 到 16 秒），所以它在几乎不重复的环路里游荡，而不是沿一条线来回走 |

## 生产注意事项
- **滤镜性能才是坑。**SVG 滤镜每帧在 CPU/GPU 合成器上光栅化滤镜区域；大视口上的大片模糊区域会让手机掉帧。把过滤的 `<g>` 区域保持在设计允许的最小范围，封顶 `stdDeviation`，小屏幕上减少色块数量。不要只在桌面上测，要在中端手机上测。
- **Safari 和模糊单位。**`stdDeviation` 用的是用户（viewBox）单位，不是像素——视觉模糊随 `viewBox` 缩放，所以固定 viewBox 大小，让 CSS 缩放 SVG，而不是调整 viewBox。
- **按时间，不按帧**：每帧弥合固定占比的缓动会跟着屏幕的刷新率走。像上面那样按自上一帧以来的时间缩放占比，并封顶这个时间（演示用 50ms），让回来的标签页不至于猛地一跳。
- **SVG 元素上的 `will-change: transform`** 可以把它提升到自己的图层、变得更平滑，但要留意低端设备的内存。
- **减弱动态。**在 `prefers-reduced-motion: reduce` 下，演示开始时处于暂停状态，显示静止的色块，直到用户按下「播放」。生产环境里，绝不要让自主翻腾的形状在选择退出的用户面前一直跑：展示一个安顿好的排列。
- **替代方案 / 库等价物：**单个动态轮廓而不是元球时，插值 SVG `<path>` 的控制点（**blobmaker** / **Haikei** 这类工具导出的做法）。重型的真正 3D 粘液，**three.js** 的 MarchingCubes（元球）或片元着色器里的光线步进 SDF 能以 GPU 成本给出真正的体积。**Framer Motion** 可以补间 `path` 的 `d` 属性，做出基于路径的色块。

## 另见
- [流体模拟](../fluid-simulation/) — 同样的融化，用 WebGL 着色器画出
- [噪声驱动运动](../noise-based-motion/) — 平滑噪声让色块的轮廓起伏
- [SVG 路径动画](../svg-path-animation/) — 自己画出来的线条
- [光线步进 / SDF 场景](../ray-marching-sdf/) — 在 3D 里融在一起的形状
- [粘性菜单](../../04-micro-interactions/gooey-menu/) — 同样的粘液，用在从圆形按钮里渗出来的按钮上
