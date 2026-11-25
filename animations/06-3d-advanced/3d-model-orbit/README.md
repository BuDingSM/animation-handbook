# 3D 模型环绕

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/3d-model-orbit/)

## 它是什么
3D 模型环绕展示一个看起来结实的物体在浏览器里转动。形状以三角形存储，WebGL 用柔和的基础光、直射光和高光照亮它的每一点，所以每一面转到眼前时都显得结实。这是 3D 产品查看器的基本模式，这里不用 3D 库画出来。

## 什么时候用
- 产品配置器，3D 物体要在悬停或滚动时旋转
- 展示动态 3D 标志或抽象形状的主视觉区
- 鼠标跟随效果，物体倾斜着追踪光标
- 展示 3D 坐标系结构的教学场景

## 工作原理
一个 WebGL 程序需要三个部分：几何体（顶点数据）、着色器（GPU 程序）和变换矩阵。

**顶点着色器**接收每个顶点的位置和法线，输出裁剪后的位置，以及传给片元着色器的插值数据：

```glsl
attribute vec3 aPos, aNorm;
uniform mat4 uMVP, uM;        /* model-view-projection, model-view */
varying vec3 vN, vP;

void main() {
  gl_Position = uMVP * vec4(aPos, 1.0);
  vP = (uM * vec4(aPos, 1.0)).xyz;
  vN = normalize(mat3(uM) * aNorm);
}
```

**片元着色器**应用 Phong 光照——环境光 + 漫反射 + 高光：

```glsl
precision mediump float;
varying vec3 vN, vP;
uniform vec3 uCol, uLight;

void main() {
  vec3 L = normalize(uLight - vP);
  vec3 V = normalize(-vP);
  float d = max(dot(vN, L), 0.0);
  float s = pow(max(dot(reflect(-L, vN), V), 0.0), 48.0);
  gl_FragColor = vec4(0.15 * uCol + d * uCol + 0.7 * s, 1.0);
}
```

**变换管线**——MVP 矩阵链：

```js
const P   = perspective(Math.PI/4, width/height, 0.1, 20);
const T   = translate(0, 0, -dist);
const M   = multiply(multiply(T, rotateY(ry)), rotateX(rx));
const MVP = multiply(P, M);
```

**转动**——角度按流逝的时间增长，而不是每帧固定增加，所以形状在 30、60 或 144 Hz 的屏幕上都以同样的速度转动：

```js
const FRAME = 1000 / 60;                     // one frame on a 60 Hz screen, in milliseconds
const step = Math.min(elapsed, 50) / FRAME;  // 1 at 60 Hz; a long gap counts for at most 50 ms
ry += SPEED_Y * 0.01 * step;                 // the spin
rx += SPEED_X * 0.01 * step * (Math.sin(clock * 0.0003) * 0.5 + 0.5);  // the forward roll, at a speed that rises and falls
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 形状 | 立方体 | 要画的物体：立方体、球体、圆环，或由四个三角形组成的金字塔 |
| 运动方式 | 自己旋转 | 自己旋转是它自己转动；跟随指针是它转向你的指针或手指，指针在舞台边缘时左右最多 86°、上下最多 69° |
| 速度 | 普通 | 它转得多快：慢大约每秒 17°，普通 27°，快 45°，在任何屏幕上都一样；翻滚时，向前翻滚的速度在约 21 秒里于静止和四分之一转速之间起伏 |
| 边转边翻滚 | 开 | 还会向前翻滚，速度时快时慢，让顶部和底部露出来 |
| 距离 | 中等 | 镜头离形状中心多远：近大约是形状宽度的 2.3 倍，中等 2.7 倍，远 3.8 倍，所以近看起来更大；镜头不变，从上到下是 45° 的视野 |
| 颜色 | 蓝色 | 物体的颜色；光照会加上阴影和高光 |
| 只显示边 | 关 | 画出组成形状的三角形的边，而不是打光的表面 |

## 生产注意事项
- **生产环境用 Three.js**：Three.js 的 `PerspectiveCamera`、`MeshPhongMaterial` 和 `OrbitControls` 用约 20 行就能替代演示里的全部内容。比单个物体更复杂的场景就用 Three.js——原生 WebGL 很快会变得难以维护。
- **按时间，不按帧**：每帧固定转一点，在 120 Hz 的屏幕上会转两倍快，在 30 Hz 的手机上慢一半。按自上一帧以来的时间缩放每次转动，封顶这个时间（演示用 50ms），让回来的标签页不至于猛地一跳，并让回来的标签页从下一帧开始。
- **法线矩阵**：严格来说，法线应该用模型矩阵的逆转置来变换，而不是模型矩阵本身。均匀缩放（没有不均匀的压扁/拉伸）时，模型矩阵够用。非均匀缩放时用 `mat3(transpose(inverse(uM)))`。
- **`prefers-reduced-motion`**：停掉动画循环，让物体以默认朝向展示。物体应保持可见，并可供镜头控制。
- **触摸/指针交互**：拖动旋转用 `pointerdown` / `pointermove` / `pointerup`，不用 `mousedown`——这样无需另外的事件监听就能覆盖触摸。
- **大声报错**：编译失败的着色器只会画出黑画布，不抛异常，因为只有主动去问才能读到 `COMPILE_STATUS`。检查它（还有 `LINK_STATUS`）并显示一条消息；无声的黑盒子是最难诊断的 WebGL bug。
- **上下文丢失**：GPU 重置、驱动更新或标签页恢复时，浏览器可能丢掉 WebGL 上下文，画布从此一直空白。生产环境要监听 `webglcontextlost`（在上面调用 `preventDefault()`），并在 `webglcontextrestored` 里重建缓冲、纹理和程序。演示为了让渲染路径可读，没有这样做。

## 另见
- [滚动驱动 3D 旋转](../scroll-driven-3d-rotation/) — 滚动转动物体，而不是时间
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张平面卡片朝指针倾斜
- [WebGL 着色器动画](../webgl-shader-animation/) — 同样的 WebGL 设置，画的是图案而不是形状
