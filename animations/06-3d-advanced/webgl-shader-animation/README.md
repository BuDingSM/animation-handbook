# WebGL 着色器动画

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/webgl-shader-animation/)

## 它是什么
着色器动画完全在显卡上画出动态画面。一个小程序每帧为每个像素运行一次，按像素的位置和时间算出它的颜色，所以整个图案都是数学做出来的，没有图片。演示里的四种图案各是这样一个程序。

## 什么时候用
- 需要 GPU 级性能的全画布动态背景
- 图案从数学里产生的生成艺术和创意编码
- 零素材开销的加载画面和过渡
- 用程序化颜色和运动表达品牌

## 工作原理
整个设置就是一个盖住屏幕的四边形。片元着色器以 uniform 接收像素坐标和全局时间：

```js
// Minimal WebGL boilerplate for a shader demo
const prog = createProgram(gl,
  `attribute vec2 a; void main() { gl_Position = vec4(a, 0., 1.); }`,
  fragmentShaderSource
);
const buf = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buf);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1,-1, 1,-1, -1,1, 1,1]), gl.STATIC_DRAW);
gl.enableVertexAttribArray(aPos);
gl.vertexAttribPointer(aPos, 2, gl.FLOAT, false, 0, 0);

let t = 0, last = 0;
function render(ts) {
  if (last) t += Math.min(ts - last, 100) / 1000 * speed;   // add up time, so a new speed never jumps
  last = ts;
  gl.uniform2f(uRes, W, H);
  gl.uniform1f(uT, t);
  gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
  requestAnimationFrame(render);
}
```

**等离子**——最简单的图案；UV 坐标上的嵌套正弦函数，它们的和挑出一个色相（`hsv()` 把色相、饱和度和亮度变成颜色）：

```glsl
void main() {
  vec2 uv = gl_FragCoord.xy / uRes * 2.0 - 1.0;
  uv.x *= uRes.x / uRes.y;                       // keep the rings round on a wide stage
  float v  = sin(uv.x * 5.0 + uT)
           + sin(uv.y * 5.0 + uT * 0.7)
           + sin((uv.x + uv.y) * 5.0 + uT * 0.5)
           + sin(length(uv) * 6.0);              // rings around the middle
  float hue = fract(v * 0.25 + uT * 0.05);       // the sum picks the hue; time slides it round the rainbow
  gl_FragColor = vec4(hsv(hue, 0.9, 0.95), 1.0);
}
```

**细胞（Voronoi）**——用网格里最近邻距离做的细胞图案；每个细胞里的点随时间慢慢漂移：

```glsl
float voronoi(vec2 p) {
  vec2 i = floor(p), f = fract(p);
  float md = 8.0;
  for (int y = -1; y <= 1; y++) for (int x = -1; x <= 1; x++) {
    vec2 n = vec2(x, y);
    vec2 pt = n + rand2(i + n + uT * 0.01);   // a random point in each cell, moving with time
    md = min(md, length(pt - f));
  }
  return md;
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 图案 | 等离子 | 等离子把正弦波加起来变成流动的颜色；波浪弯曲条纹状的颜色；细胞把平面分成围绕移动点的细胞；万花筒把一片围绕中心镜像 |
| 速度 | 普通 | 图案里时间流逝的速度：慢是真实时间的 0.6 倍，普通 1 倍，快是 1.6 倍，所以等离子的颜色绕彩虹一圈，慢约 33 秒，普通 20 秒，快 12.5 秒 |
| 对指针有反应 | 关 | 每种图案都以自己的方式在指针或手指周围弯曲；万花筒里指针决定切片的数量 |

## 生产注意事项
- **始终检查 `COMPILE_STATUS` 和 `LINK_STATUS`。**编译失败的着色器不抛错也不记日志——`gl.drawArrays` 只是悄悄什么也不画，你得到一块黑画布。读 `getShaderInfoLog` / `getProgramInfoLog`，把消息放到人能看到的地方。演示把编译日志渲染进舞台；故意弄坏一个着色器，你会得到 GLSL 错误，而不是黑盒子。
- **处理上下文丢失。**GPU 重置、驱动更新或操作系统回收的后台标签页会触发 `webglcontextlost`。在上面调用 `preventDefault()`——否则上下文永远回不来——然后在 `webglcontextrestored` 里重建缓冲、程序和 uniform 位置。旧上下文的每个 GL 对象都不见了。
- **按设备像素设置后备存储大小**，不是 CSS 像素：`canvas.width = clientWidth * dpr`。然后在同一个地方更新 `gl.viewport(0, 0, canvas.width, canvas.height)`——忘记视口是经典的调整大小 bug，它会拉伸或裁剪场景，而不是大声报错。
- **Shadertoy**：演示里的四种图案都是经典的 Shadertoy 练习。这个网站有实时 GLSL 编辑器、庞大的社区着色器库和标准的 uniform 约定（`iTime`、`iResolution`、`iMouse`）。
- **画布分辨率**：着色器成本随像素数缩放。4K 下复杂着色器的运行速度是 1080p 下的四分之一。谨慎使用 `devicePixelRatio`——按 0.5 倍设备像素渲染再放大，背景着色器常常看起来没问题。
- **`mediump` 与 `highp`**：手机上要求 `precision mediump float`。有些着色器在 `mediump` 下会产生可见的条带——细节精细的图案改用 `precision highp float`。
- **鼠标 uniform**：加上 `uniform vec2 uMouse`，传入 `e.clientX / W, e.clientY / H`，不用重写核心算法就能让任何着色器可交互。

## 另见
- [流体模拟](../fluid-simulation/) — 同样的单面着色器设置画出的色块
- [光线步进 / SDF 场景](../ray-marching-sdf/) — 一个着色器画出的整个 3D 场景
- [噪声驱动运动](../noise-based-motion/) — 在 2D 画布上画出的动态图案
