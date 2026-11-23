# 流体模拟

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/fluid-simulation/)

## 它是什么
这个流体效果看起来像液体，但并不是真正的流体模拟。WebGL 着色器对每个像素算出它离几个移动圆圈有多远，再把这些距离混合起来，让靠得近的圆圈融成一个平滑的形状，像水银滴或熔岩灯。漂开的圆圈又会干净地分开。

## 什么时候用
- 健康、身心健康和创意产品网站上液体美学的主视觉背景
- 生成式品牌元素，有机运动传递“活着”的感觉
- 让界面元素感觉柔软流畅的悬停效果
- 自助终端或仪表盘的屏保式氛围背景

## 工作原理
每个像素的颜色由它在世界坐标处所有元球的 SDF 决定。`smin`（平滑最小值）函数混合重叠的 SDF：

```glsl
// Smooth minimum — merges two distance fields
float smin(float a, float b, float k) {
  float h = clamp(0.5 + 0.5 * (b - a) / k, 0.0, 1.0);
  return mix(b, a, h) - k * h * (1.0 - h);
}

// Metaball scene
float scene(vec2 uv) {
  float d = 1e10;
  for (int i = 0; i < 5; i++) {
    vec2 center = ballPosition(i, uTime);  // animated with sin/cos
    float ballDist = length(uv - center) - RADIUS;
    d = smin(d, ballDist, SMOOTHNESS);    // merge softly
  }
  return d;
}

void main() {
  vec2 uv = gl_FragCoord.xy / uResolution;
  float d = scene(uv);
  float inside = smoothstep(0.01, -0.01, d);  // binary in/out
  gl_FragColor = vec4(mix(BG_COLOR, FLUID_COLOR, inside), 1.0);
}
```

圆球的位置按参数化的正弦/余弦曲线运动，保证运动连续、感觉不重复：

```glsl
vec2 ballPosition(int i, float t) {
  float fi = float(i);
  float s = 1.3 + fi * 0.4;
  float o = fi * 2.1;
  return vec2(0.5 + 0.38 * sin(t * s + o),
              0.5 + 0.38 * cos(t * s * 0.7 + o * 1.3));
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 融化程度 | 中等 | 两个色块相隔多远时开始融化到一起：一点是舞台高度的 0.03，中等 0.08，很多是 0.15；很多会形成粗而柔软的颈部 |
| 色块数量 | 中等 | 少是 3 个，中等 5 个，多是 8 个色块；每个色块都会给每个像素增加工作量 |
| 速度 | 普通 | 色块沿路径移动的速度：慢时每个色块来回摆动一次要 3 到 10 秒，普通 2 到 6 秒，快 1.2 到 3.7 秒，各自的节奏不同 |
| 色块大小 | 中等 | 每个色块的半径：小是舞台高度的 9%，中等 14%，大是 20% |
| 颜色 | 蓝色 | 液体的颜色 |
| 辉光 | 开 | 在液体周围加一圈渐渐淡出的柔和光晕 |
| 一个色块跟随指针 | 开 | 一个色块离开自己的路径，停在指针或手指下面 |

## 生产注意事项
- **真正的流体模拟**：基于 Navier-Stokes 的流体（速度场、压力、扩散）需要每帧更新全屏纹理。Pavel Dobryakov 的 WebGL Fluid Simulation（GitHub 上的 PavelDoGreat，开源）是首选——它用一系列物理通道以 60fps 渲染真正可交互的流体。
- **SDF 元球只是近似**：它们看起来像流体，但不守恒体积，不会绕过障碍物，也不会响应物理力。要真正的流体行为，用完整的模拟库。
- **性能**：每多一个元球，每个像素就多一次距离计算。1920×1080 的画布上有 8 个球，一帧就是约 1600 万次 SDF 计算。这就是手机帧率下降的原因——手机上用更低的画布分辨率。演示把后备存储封顶在桌面 2 倍设备像素比、手机 1.5 倍，这就是它全部的质量/成本调节。
- **无声的着色器失败**：编译失败的着色器不抛错也不记日志——你只会得到黑画布。始终检查 `COMPILE_STATUS` 和 `LINK_STATUS` 并把消息显示出来，演示就是这样做的。
- **上下文丢失**：GPU 重置、驱动更新或恢复后台标签页时会触发 `webglcontextlost`。没有监听器，画布会永久变黑。在丢失事件上调用 `preventDefault()`，停掉循环，并在 `webglcontextrestored` 里重建程序。
- **Shadertoy**：元球图案是经典的 Shadertoy 练习之一。[shadertoy.com](https://www.shadertoy.com) 上有几百种元球变体。
- **`prefers-reduced-motion`**：暂停动画。色块应停在默认位置保持可见。

## 另见
- [WebGL 着色器动画](../webgl-shader-animation/) — 同样的单面着色器设置，画的是图案
- [变形色块](../morphing-blob/) — 同样的融化外观，用模糊的圆圈做出，不用 WebGL
- [噪声驱动运动](../noise-based-motion/) — 平滑噪声移动圆点和色块
