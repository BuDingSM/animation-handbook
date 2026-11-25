# 悬停图片扭曲

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/image-distortion-hover/)

## 它是什么
悬停图片扭曲在指针周围弯曲图片。WebGL 着色器画出图片，指针附近的每个像素从略微偏移的位置读取，所以图片在那里像是起涟漪、鼓起来、漩涡转动或碎成方块，而图片本身从不改变。指针离开后弯曲慢慢消失。

## 什么时候用
- 代理公司作品集网格，每张项目卡片悬停时扭曲，表明可以交互
- 对光标有反应、带微妙液体动画的主视觉图片
- 浏览时感觉“活着”的产品展示瓷贴
- 任何需要鼠标交互、又不想用大量 JavaScript 动画的静态图片

## 工作原理
核心操作是片元着色器里的 UV 位移。UV 坐标 `uv` 处的像素在 `uv + displacement(uv, mouseUV)` 处采样纹理：

```glsl
// Ripple distortion: concentric waves from mouse position
vec2 ripple(vec2 uv, vec2 mouseUV, float strength, float radius, float phase) {
  vec2 dir = uv - mouseUV;
  float dist = length(dir);
  float falloff = smoothstep(radius, 0.0, dist);   // fade with distance
  float wave = sin(dist / radius * 12.0 - phase) * falloff * strength;
  return normalize(dir) * wave;                     // offset along the ray
}

void main() {
  vec2 uv = gl_FragCoord.xy / uResolution;
  vec2 mouseUV = uMousePos / uResolution;

  vec2 distortedUV = uv + ripple(uv, mouseUV, 0.04, 0.25, uPhase);
  gl_FragColor = vec4(sampleTexture(distortedUV), 1.0);
}
```

`uPhase` uniform 每帧前进，让涟漪即使在鼠标停下后仍然向外扩散。真正淡出的是另一项**振幅**：指针在舞台上时它固定为 1，指针离开后指数衰减，所以扭曲会平滑回到未位移的图片，而不是一下子关掉。

```js
// Phase always advances; amplitude decays when the pointer is gone
phase += dt * 0.003;
if (!pointerActive) amp *= Math.max(0, 1 - DECAY * dt / 1000);
gl.uniform1f(uPhase, phase);
gl.uniform1f(uStrength, STRENGTH * amp);
```

**程序化纹理**（不需要外部图片）可以在同一个着色器里生成图案：

```glsl
vec3 pattern(vec2 uv) {
  float s = sin(uv.x * 8.0 + uTime) * sin(uv.y * 8.0 + uTime * 0.7);
  return mix(vec3(0.2, 0.35, 0.6), vec3(0.6, 0.2, 0.7), s * 0.5 + 0.5);
}
```

演示里图案是静止的，和真实图片一样：它的时间值保持在 0，只有扭曲在动。指针离开、淡出完成后，演示停下绘制，指针或「演示一下」会重新启动它。

图片也可以获得键盘焦点，方向键代替指针：键盘焦点在图片中间（或上次弯曲的地方，正在淡出时）弯曲图片，每按一次方向键，那个点就向该方向移动图片宽度的 6%（按住会重复移动）。按 Escape 或把焦点移走会让弯曲淡出，和指针离开时一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 效果 | 涟漪 | 涟漪从指针处向外送出圆环；推开让图片向外鼓起；液体让它漩涡转动；像素块把它碎成方块 |
| 强度 | 中等 | 图片弯曲的程度：涟漪在轻柔时最多移动宽和高的 2%，中等 4%，强烈 7%；推开移动 1.6 倍，液体 0.6 倍，像素块做出的方块最大是图片宽和高的 1%、1.7% 或 2.9%。越强看起来越夸张，但越难读 |
| 大小 | 中等 | 弯曲的区域有多宽：指针周围的一个圆，半径在小是图片高度的 15%，中等 25%，大是 40%；弯曲从指针处的全额渐变到边缘处的零 |
| 消退速度 | 普通 | 指针离开后弯曲消失的速度：慢时每秒失去约五分之三的强度，普通约五分之四，快是十分之九 |
| 涟漪扩散 | 开 | 涟漪时，指针停着不动，圆环仍然继续向外扩散；关掉就静止不动 |

## 生产注意事项
- **真实图片**：把程序化的 `pattern()` 函数换成 `texture2D(uTexture, distortedUV)`。用 `gl.texImage2D()` 从 `<img>` 或 `ImageBitmap` 把图片载入 WebGL。同源策略适用——外部图片 URL 需要 CORS 头。
- **`CLAMP_TO_EDGE`**：把纹理的包裹模式设为 `gl.CLAMP_TO_EDGE`（`TEXTURE_WRAP_S` 和 `TEXTURE_WRAP_T` 都要），这样扭曲把 UV 推出范围时，图片边缘不会平铺或镜像。
- **用指针，不用鼠标**：绑定 `pointermove`/`pointerdown` 而不是 `mousemove`，否则手机上效果永远不会触发。光标位置在到达着色器之前必须换算成后备存储像素（乘以画布用的设备像素比），否则高 DPI 屏上扭曲中心会偏离手指。
- **后备存储和 DPR**：画布按 `clientWidth * dpr` 设置大小，每次调整大小后调用 `gl.viewport()`。不封顶的 DPR 对全屏片元着色器来说很贵——演示封顶在 2，手机上 1.5。调整大小会清空绘制缓冲，所以不每帧绘制的演示（这个在静止时停下）必须在调整大小后重绘，否则会变黑。
- **上下文丢失**：GPU 重置或恢复的标签页会触发 `webglcontextlost`。没有监听器（以及 `preventDefault()`，这样 `webglcontextrestored` 才会跟上），画布会永久死亡。恢复时着色器、缓冲和 uniform 位置都必须重建。
- **纯 CSS 替代**：伪元素上的 CSS `filter: blur()` 和 `transform: translate()` 可以在低强度下近似推开扭曲，针对单个元素。逐像素的波浪和液体效果需要 WebGL。
- **hover-effect-curtains / Curtains.js**：包装了这个确切模式的生产库。它们处理纹理加载、画布大小和着色器样板。GLSL 片元着色器与演示用的完全相同。
- **Shader Park**：用类似 JavaScript 的语法声明扭曲效果、再编译成 GLSL 的高层工具。适合写原生 GLSL 有门槛的创意应用。

## 另见
- [WebGL 着色器动画](../webgl-shader-animation/) — 同样的着色器设置，画的是图案
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜，而不是弯曲
- [流体模拟](../fluid-simulation/) — 跟随指针的液体外观
