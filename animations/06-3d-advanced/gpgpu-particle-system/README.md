# GPGPU 粒子系统

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/gpgpu-particle-system/)

## 它是什么
GPU 粒子系统把数万个粒子放在显卡上移动，而不是在 JavaScript 里。每个粒子的位置和速度存为纹理的一个像素，着色器读取上一帧的纹理、同时写出下一帧。第二步把每个粒子画成一个小点。

## 什么时候用
- 大规模粒子效果：星系模拟、流体流动、火焰系统、人群模拟
- 任何 JavaScript 粒子数量成为性能瓶颈的粒子系统
- 粒子数量本身就是美学的创意编码场景（10 万个点拼成一个形状）
- 浏览器里 GPGPU 原理的技术演示

## 工作原理
**数据布局**：一张 256×256 的 RGBA32F 像素纹理存储 65536 个粒子。每个像素 = 一个粒子的 `(x, y, vx, vy)`。

**物理着色器**（WebGL2 片元着色器，渲染到离屏纹理）：

```glsl
#version 300 es
precision highp float;
uniform sampler2D uPos;   // read previous frame's positions
uniform float uTime, uSpeed;
uniform int uBehavior;
uniform float uStep;     // one 60 Hz step per pass: 1, or 1/3 in slow motion
out vec4 outColor;        // write new positions

float hash(float n) { return fract(sin(n) * 43758.5); }

void main() {
  ivec2 coord = ivec2(gl_FragCoord.xy);
  vec4 p = texelFetch(uPos, coord, 0);
  float x = p.x, y = p.y, vx = p.z, vy = p.w;

  // Apply behavior (flow field, vortex, attractor, etc.)
  if (uBehavior == 1) {  // vortex
    float len = length(vec2(x, y)) + 0.01;
    vx += (-y / len) * uSpeed * 0.0004 * uStep;
    vy += ( x / len) * uSpeed * 0.0004 * uStep;
    vx -= x * 0.0005 * uStep;  // inward pull to prevent escape
    vy -= y * 0.0005 * uStep;
  }
  vx *= pow(0.985, uStep); vy *= pow(0.985, uStep);  // damping
  x += vx * uStep; y += vy * uStep;
  // Respawn anything that leaves clip space
  if (abs(x) > 1.0 || abs(y) > 1.0) {
    float n = float(coord.x) * 0.0713 + float(coord.y) * 0.1319 + uTime * 0.977;
    float a = hash(n) * 6.2832, r = hash(n + 3.71) * 0.9 + 0.05;
    x = cos(a) * r; y = sin(a) * r; vx = 0.0; vy = 0.0;
  }
  outColor = vec4(x, y, vx, vy);
}
```

**时间步长**——每次物理通道都让粒子恰好移动一个 60 Hz 步长：`uStep` 是 1（慢动作里是它的三分之一），在任何屏幕上都一样。页面查看屏幕提供的每一帧，按六十分之一秒的步数测量自上次绘制以来的时间（一个间隔最多算 50ms，与整数步数相差 0.05 以内的值算作那个整数，所以略微偏离 60 Hz 的屏幕，比如 59.94 或 119.88 Hz，每帧仍然恰好跑一次通道）。它为累加起来的每个完整步数跑一次通道（留 1ms 宽限，每帧最多三次），剩下的带到下一帧：60 Hz 的屏幕每帧一次，30 Hz 的两次，来得太早的一帧一次也不跑，什么也不改、什么也不画，所以更快的屏幕上绘制保持在每秒 60 张图附近。粒子在任何屏幕上移动的速度都一样，吸引器（在中间附近拉得狠，长步和短步的落点不同）看起来也一样。绘制的一帧在通道之后为跑过的每个通道在画面上铺一次拖影淡出，每次是旧画面的 12%；慢动作每秒仍然跑 60 次通道，所以它让粒子变慢，但不让淡出变慢。一次更小的淡出淡掉的量理论上相同，但 8 位画布每次覆盖都会四舍五入，拖影淡淡的残留最后会变成另一个颜色。

**纹理乒乓**——每次通道交替把哪张纹理作为读取源：

```js
// Physics pass: read tex0, write to fbo1 (which wraps tex1)
gl.bindFramebuffer(gl.FRAMEBUFFER, fbo1);
gl.bindTexture(gl.TEXTURE_2D, tex0);
gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

// Swap
[tex0, tex1] = [tex1, tex0];
[fbo0, fbo1] = [fbo1, fbo0];
```

**渲染通道**——顶点着色器读取位置纹理来设置 `gl_Position`：

```glsl
#version 300 es
in float aIndex;
uniform sampler2D uPos;
uniform vec2 uTexSize;
uniform float uPointSize;   // 1.5 CSS px x devicePixelRatio

void main() {
  int idx = int(aIndex);
  ivec2 co = ivec2(int(mod(aIndex, uTexSize.x)), idx / int(uTexSize.x));
  vec4 p = texelFetch(uPos, co, 0);
  gl_Position = vec4(p.xy, 0.0, 1.0);
  gl_PointSize = uPointSize;
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 运动方式 | 流动 | 流动跟随一片变化的水流；漩涡绕着中间转；吸引把一切拉向中间；环绕让粒子成圈扫过 |
| 粒子数量 | 65,000 | 16000、65000 或 100000 个粒子，各是一张 128、256 或 320 像素见方纹理的一个像素；手机从 16000 个开始 |
| 速度 | 普通 | 每六十分之一秒规则推的力度：慢是快的 0.35 倍，普通是 0.6 倍；流动时一个典型的粒子每秒横穿舞台约 11%、16% 或 22% |
| 颜色 | 单色 | 一种蓝色、按每个粒子速度着色，或慢慢变化的颜色 |
| 拖影 | 关 | 淡出上一张画面而不是清除它，让每个粒子留下一道拖影 |

## 生产注意事项
- **需要 WebGL2**：`RGBA32F` 浮点帧缓冲目标需要 WebGL2（或 WebGL1 里的 `WEBGL_color_buffer_float` 扩展，可靠性较差）。所有现代浏览器都支持 WebGL2（Chrome 56+、Firefox 51+、Safari 15+）。
- **按时间，不按帧**：每帧固定一步会让粒子在 120 Hz 的屏幕上移动两倍快，在 30 Hz 的手机上慢一半，拖影也以同样错误的速率淡出。像上面那样从累加器里按固定 1/60 秒走步，封顶一帧的时间（演示用 50ms）和每帧的通道数，让靠近一点的强拉力在任何屏幕上落点相同。每物理通道淡出一次，始终淡出同样的量。
- **Three.js 的 `GPUComputationRenderer`**：Three.js 自带 `GPUComputationRenderer`（在 `three/examples/jsm` 路径下），把整个乒乓纹理模式封装起来。这是惯用的生产做法。
- **WebGPU 计算着色器**：WebGPU 提供真正的 `@compute` 着色器阶段给 GPGPU 用，省掉了“用片元着色器做计算”这个变通办法。截至 2024 年，WebGPU 在 Chrome 113+ 里可用，Firefox 正式版和 Safari 里还没有。
- **只有无散场才能用环绕**：演示的流场把粒子从 x 轴向外推，所以有个向外的净漂移。早期版本在 ±1.2 处环绕，而可见的只有 ±1.0；粒子漂进那个看不见的环的速度比回来快，画布约十秒后就淡成黑色。把环绕收紧到 ±1.0 只是挪了症状——有散度的场会把逃出去的粒子直接传送回它刚离开的边缘，于是粒子反而沿着顶部和底部结壳。把逃出去的粒子在中间附近的随机点重生才是修复办法，也是这里让数量保持稳定的原因。两种失败都不会抛错，头一秒看起来也都正确，所以这类 bug 只有让演示一直跑着才能看见。
- **点的大小以设备像素计**：画布后备存储按 `clientWidth * devicePixelRatio`（这里封顶 2）设置，`gl.viewport` 用同样的数字。`gl_PointSize` 也是以设备像素计的，所以也要乘以同样的比例，否则点在视网膜屏上会变小。
- **上下文丢失**：GPU 重置、标签页恢复或太多活动的 WebGL 上下文都会触发 `webglcontextlost`。没有调用 `preventDefault()` 的监听器，上下文永远回不来，演示会一直黑下去。在 `webglcontextrestored` 里每个程序、缓冲、纹理和帧缓冲都必须重建——GPU 对象在丢失后无法存活。
- **手机预算**：65000 个加色混合的点能把手机屏幕盖好几遍，又慢、视觉上又糊。演示在手机（宽 600px 及以下，或高 500px 及以下）上降到 128×128 纹理（16000 个粒子）；生产系统应该按实际要填充的像素数缩放纹理大小。
- **内存**：256×256 × 4 通道 × 4 字节（float32）= 每张纹理 1MB。两张纹理 = 2MB——微不足道。512×512（262k 个粒子）时：4MB × 2 = 8MB。

## 另见
- [Canvas 粒子效果](../canvas-particle-effect/) — 在 JavaScript 里移动的几百个粒子
- [流体模拟](../fluid-simulation/) — 把色块融在一起的着色器
- [WebGL 着色器动画](../webgl-shader-animation/) — 逐像素作画的着色器
