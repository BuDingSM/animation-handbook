# 光线步进 / SDF 场景

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/ray-marching-sdf/)

## 它是什么
光线步进不用任何 3D 模型就能画出 3D 场景。每个形状都是一个给出到它表面距离的公式，对每个像素，着色器按那个距离让一条光线向前走，直到碰到东西，再照亮那个点。因为形状是公式，它们可以融在一起、互相挖空，或永远重复。

## 什么时候用
- 难以用网格建模的复杂动态 3D 形状（分形、布尔混合、有机变形）
- 全屏着色器艺术和生成式视觉
- 不用 Three.js 开销的 3D 数学教学演示
- 浏览器里 Shadertoy 风格的创意编码

## 工作原理
**核心光线步进循环**——让一条光线前进，直到它离表面只有一个极小值：

```glsl
float rayMarch(vec3 ro, vec3 rd) {  // ray origin, ray direction
  float t = 0.0;
  for (int i = 0; i < MAX_STEPS; i++) {
    float d = scene(ro + rd * t);   // distance to nearest surface
    if (d < 0.001) return t;        // hit
    t += d * 0.9;                   // step a little short: blended and carved shapes are not exact distances
    if (t > 22.0) break;            // escaped scene
  }
  return -1.0;                       // no hit
}
```

**SDF 图元**——基本构件：

```glsl
float sdSphere(vec3 p, float r) { return length(p) - r; }

float sdBox(vec3 p, vec3 b) {
  vec3 q = abs(p) - b;
  return length(max(q, 0.0)) + min(max(q.x, max(q.y, q.z)), 0.0);
}

float sdTorus(vec3 p, vec2 t) {
  vec2 q = vec2(length(p.xz) - t.x, p.y);
  return length(q) - t.y;
}
```

**平滑最小值**混合两个 SDF，产生有机的融合效果：

```glsl
float smin(float a, float b, float k) {
  float h = clamp(0.5 + 0.5 * (b - a) / k, 0.0, 1.0);
  return mix(b, a, h) - k * h * (1.0 - h);
}
```

**用有限差分求法线**——在 6 个方向上采样 SDF：

```glsl
vec3 getNormal(vec3 p) {
  vec2 e = vec2(0.001, 0.0);
  return normalize(vec3(
    scene(p + e.xyy) - scene(p - e.xyy),
    scene(p + e.yxy) - scene(p - e.yxy),
    scene(p + e.yyx) - scene(p - e.yyx)
  ));
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 场景 | 融合 | 融合把一个球、一个方块和一个圆环融在一起；挖空从一个球里挖出一个方块，里面还有一个小球；无尽复制让一个挖空的方块在地板上无尽重复 |
| 镜头速度 | 普通 | 镜头绕形状转一圈的速度：慢约 18 秒，普通 10 秒，快 6 秒 |
| 柔和阴影 | 关 | 从每个表面点向光源再投一条光线，做出柔和阴影；最耗性能的选项 |
| 角落阴影 | 开 | 在每个表面上方测试几个点，让表面相接的折痕和角落变暗 |
| 细节 | 中等 | 一条光线最多走多少步：低是 24 步，中等 48 步，高是 96 步；手机上是三分之二 |
| 步数视图 | 关 | 按每条光线走的步数给每个像素上色：边缘和差点碰到的地方步数最多 |

## 生产注意事项
- **Shadertoy 约定**：uniform 是 `iTime`、`iResolution`、`iMouse`。把 Shadertoy 代码移植到 WebGL 需要把它们改成自己的 uniform 名，并加上 WebGL 样板（顶点着色器 + 四边形）。
- **性能随像素和每步的工作量缩放**：每条光线的每步都会计算整个场景公式，所以每多一个形状或运算都会加到每步上，4K 显示器的成本是 1080p 的 4 倍。演示把画布封顶在约 300000 像素，让浏览器把它放大；手机上还按舞台的 0.7 绘制，步数取三分之二。
- **柔和阴影和 AO**：两者每个片元都需要额外的光线（阴影光线、AO 采样）。柔和阴影很贵——每个受光像素 20 步阴影步进会让总步进工作量翻倍。低端设备上关掉。
- **`mediump` 精度**：带大尺度重复的复杂距离函数在 `mediump` 下会丢失精度。带重复图案或精细几何的 SDF 场景用 `highp float`。
- **Three.js 替代方案**：带 `ShaderMaterial` 的 Three.js 把同样的 uniform 传给同样的片元着色器——GLSL 完全相同，只有 WebGL 样板不同。

## 另见
- [体积烟雾 / 3D 噪声](../volumetric-smoke/) — 累加烟雾而不是停在表面的光线
- [WebGL 着色器动画](../webgl-shader-animation/) — 同样设置上的更简单着色器
- [流体模拟](../fluid-simulation/) — 同样的形状融化，在 2D 里
