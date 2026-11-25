# 体积烟雾 / 3D 噪声

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/volumetric-smoke/)

## 它是什么
体积烟雾是按真正的 3D 云画出来的，而不是一张平面图。对每个像素，一条光线穿过一片平滑噪声的云，累加它遇到的烟雾量，再按它与光源之间的烟雾量给每个部分上色。这带来柔和的边缘和深度，电影就是这样画烟雾、雾和云的。

## 什么时候用
- 氛围背景效果：上升的烟雾、体积雾、云的形态
- 体积薄雾营造氛围的科幻或奇幻环境
- 密度/体积是视觉隐喻的数据可视化
- 带环境效果的动态标题卡和主视觉区

## 工作原理
对每个像素，一条光线在 3D 空间里步进。每步采样噪声场的密度并累加。光衰减通过向光源投一条短阴影光线算出：

```glsl
void main() {
  vec2 uv = (gl_FragCoord.xy - 0.5 * uResolution) / min(uResolution.x, uResolution.y);
  vec3 ro = vec3(0., 0., 3.);        // camera origin
  vec3 rd = normalize(vec3(uv, -0.9)); // ray direction

  float transmittance = 1.0;
  vec3 color = vec3(0.0);            // background is composited once, at the end
  float stepSize = 3.0 / float(uSteps);

  for (int i = 0; i < MAX_STEPS; i++) {
    if (i >= uSteps) break;
    vec3 p = ro + rd * float(i) * stepSize;

    float d = smokeDensity(p, uTime);  // sample noise at this 3D point
    if (d > 0.0) {
      // Shadow ray: march toward light
      float shadow = 1.0;
      for (int s = 1; s <= 3; s++) {
        shadow *= exp(-smokeDensity(p + lightDir * float(s) * 0.18, uTime) * stepSize * 2.5);
      }
      float alpha = 1.0 - exp(-d * stepSize * 8.0);
      color += transmittance * alpha * uSmokeColor * (0.15 + 0.85 * shadow);
      transmittance *= (1.0 - alpha);
    }
    if (transmittance < 0.01) break;
  }
  gl_FragColor = vec4(color + uBackground * transmittance, 1.0);
}
```

**3D 噪声密度函数**——上升烟雾形状上的 fBm：

```glsl
float smokeDensity(vec3 p, float t) {
  float r = length(p.xz);
  // smoothstep needs edge0 < edge1, so the upper falloff is inverted rather than reversed
  float src = exp(-r * 3.0) * smoothstep(-0.2, 0.6, p.y) * (1.0 - smoothstep(1.0, 1.8, p.y));
  if (src < 0.001) return 0.0;  // nothing to add here, so skip the noise
  float n = fbm(p * 1.4 + vec3(windDrift, rise, 0.0));
  return max(0.0, n - 0.45) * src * density * 2.0;
}
```

`rise` 每帧按该帧的时间 × 速度增长，在页面上算，所以改变速度永远不会让烟雾跳动。

`src` 淡到几乎没有的地方，函数在噪声之前就返回了，所以五次噪声采样只发生在可能有烟雾的地方。再加上“手机成本”里说的更小的画布，演示在笔记本和手机上都保持流畅。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 粗细 | 中等 | 烟雾有多浓：稀薄是中等的 0.6 倍，浓厚是 1.7 倍，所以最浓的烟缕在稀薄时遮住身后约三分之一，中等时一半，浓厚时 70%；稀薄看起来缥缈 |
| 速度 | 普通 | 烟雾上升的速度：图案每秒移动盒子高度的约 3%（慢）、5%（普通）和 9%（快） |
| 细节 | 中等 | 每条光线走多少步：低是 16 步，中等 32 步，高是 64 步（手机上 48 步）；步数越少，条纹越明显 |
| 烟雾颜色 | 灰蓝色 | 被照亮的烟雾的颜色；阴影部分是它的深色 |
| 风 | 开 | 烟雾慢慢向一侧飘，每秒约盒子高度的 2%，烟柱留在原地 |
| 细颗粒 | 开 | 每条光线从略微随机的点开始，把步数少留下的条纹变成细颗粒 |

## 生产注意事项
- **精度**：基于 `sin` 的哈希乘以 43758.5，会溢出 16 位 `mediump` 浮点数的可用范围，在许多手机 GPU 上把噪声塌成平块。在片元着色器里请求 `highp`（用 `GL_FRAGMENT_PRECISION_HIGH` 守卫），或把哈希换成乘数更小的不用整数的变体。
- **手机成本**：这是填充率受限的——每个片元最多跑 `步数 ×（fbm + 3 次阴影采样）`，所以演示从三方面削减。`smokeDensity` 在烟柱淡出的地方会在噪声之前返回。后备存储封顶在约 65000 像素，浏览器把它放大到舞台，柔和的烟雾正适合；每个片元也绝不超过 1 个 CSS 像素。手机上（宽 600px 及以下，或横持时高 500px 及以下）后备存储还最多是舞台的 0.75，步进最多 48 步。按 3 倍手机 DPR 画会让成本变成九倍。
- **上下文丢失**：GPU 重置、手机上的后台标签页或驱动故障会触发 `webglcontextlost`。没有监听器，画布会永久变黑且没有报错。阻止默认事件、在 `webglcontextrestored` 里重建着色器和缓冲是生产路径；演示走了更简单的路：停掉循环并显示一条消息。
- **步数与质量**：每多一步步进，像素成本线性增加。柔和的烟雾一般 32 步就够了；只有主视觉质量的渲染才用 64 步。加上蓝噪声抖动，随机偏移每条光线的起点，打散低步数下的条带瑕疵。偏移必须是*整步*的一小部分才有用——抖动几个千分之一单位是看不见的。
- **阴影光线**：3 步阴影步进给每个受光像素增加 3 倍的密度采样。低端设备上关掉。生产环境预先计算体素化的阴影图再采样。
- **实时火焰和烟雾**：游戏引擎用带加色混合精灵图的粒子系统做火焰/烟雾，比光线步进快。光线步进的体积用在离线渲染（电影、VFX）和电影级游戏过场里。
- **Three.js 的 `FogExp2`**：简单的大气用 Three.js 内置的指数雾，计算上免费（顶点着色器里按深度应用的雾）。只有雾必须有特定 3D 形状时才用体积光线步进。
- **时间累积**：实时体积引擎（虚幻引擎的 Volumetric Fog）把光线步进采样分散到多帧再混合结果。每帧成本降低 4–8 倍，代价是快速镜头运动时有轻微的重影瑕疵。

## 另见
- [光线步进 / SDF 场景](../ray-marching-sdf/) — 停在表面而不是累加烟雾的光线
- [噪声驱动运动](../noise-based-motion/) — 同一种噪声，在平面画布上
- [流体模拟](../fluid-simulation/) — 用着色器画出的柔和液体外观
