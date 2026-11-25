# 网格渐变

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/mesh-gradient/)

## 它是什么
网格渐变是几个大彩色圆圈的背景，模糊后边缘变软、颜色相互交融；重度模糊时它们融成柔和的色晕。每个圆圈沿自己的路径慢慢漂移、变大或缩小，所以颜色不断以新的方式交融，像永远没干透的颜料。这是 Stripe、Linear 和许多现代软件网站的样子。

## 什么时候用
- 品牌调色板必须感觉高级、手工打造的营销主视觉区
- 深色主题仪表盘，替代扁平黑色背景
- 需要温暖和深度、又不用图片的 SaaS 产品页
- 应用启动画面和加载状态

## 工作原理
每个色块是一个绝对定位的圆形 `<div>`，填一种颜色，用 `filter: blur()` 柔化。关键是重度模糊——80px 或更多——它把圆圈生硬的边缘溶解成一片色彩。每个色块有自己的关键帧，从起点到终点的一次移动，正放倒放（`alternate`），时长是 `--dur`（速度设置）各自的倍数：

```css
.blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(var(--blob-blur));
  opacity: .75;
  mix-blend-mode: var(--blend);
}

.b1 {
  width: 60%; height: 60%;
  background: #1a3060;        /* deep blue */
  top: -10%; left: -10%;
  animation: b1 var(--dur) ease-in-out infinite alternate;
}

.b2 {
  width: 55%; height: 55%;
  background: #2e0a50;        /* deep purple */
  top: 20%; right: -15%;
  animation: b2 calc(var(--dur) * 1.1) ease-in-out infinite alternate;
}

@keyframes b1 {
  0%   { transform: translate(0, 0)      scale(1); }
  100% { transform: translate(20%, 15%)  scale(1.2); }
}
@keyframes b2 {
  0%   { transform: translate(0, 0)      scale(1); }
  100% { transform: translate(-15%, 20%) scale(0.85); }
}
```

其他色块遵循同样的模式，分别是 `--dur` 的 0.9、1.2 和 0.8 倍（普通时 40 秒）。它们一起开始，但因为没有两个用时相同，它们很快就错开步调，颜色不断以新的方式相遇。

**CSS 混合模式**改变重叠色块的视觉混合：
- `normal`——色块彼此叠加；DOM 里靠后的在上面
- `screen`——相加 RGB 值，让重叠处更亮、更饱和
- `overlay`——暗部更暗、亮部更亮；高对比效果

重度模糊是手机上耗费的部分，所以演示在小屏幕上把它减半：

```css
@media (max-width: 600px) {
  .blob { filter: blur(calc(var(--blob-blur) * .5)); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 模糊程度 | 中等 | 色块模糊多少：轻微是 50px，中等 80px，强烈是 130px；低于约 80px 时圆圈可见，高于它就变成色晕。宽最多 600px 的屏幕用一半 |
| 速度 | 普通 | 一个色块单向漂移需要的时间：慢速 64 秒，普通 40 秒，快速 24 秒，每个色块是它的 0.8 到 1.2 倍 |
| 颜色 | 冷色 | 色块颜色：冷色、暖色、日落、合成波或大地色调 |
| 色块数量 | 4 | 三个、四个或五个色块；更多让混合更丰富 |
| 混合模式 | 普通 | 重叠色块如何混合：普通是叠加（正常混合），更亮让重叠处变亮（滤色），更深让它变暗、加深颜色（叠加） |

## 生产注意事项
- **GPU 上的 `filter: blur()`**：重度模糊在现代浏览器里是 GPU 加速的，但每个模糊元素创建一个合成层。四个模糊色块就是四个合成层。在低内存设备上这会累积——在手机上测试。
- **溢出裁剪**：色块超出容器。父元素一定要 `overflow: hidden`，色块才不会渗到页面的其他地方。
- **原生 CSS 网格渐变**：CSS 还没有。网格和自由形式渐变的提案在 CSS 工作组是开放的（csswg-drafts issue #7648），但它不在任何规范里，也没有浏览器实现，所以模糊色块仍然是实现这个效果的方法。
- **Figma / Sketch 的“网格渐变”工具**：设计工具把网格渐变生成为图片。网页上，模糊 div 的做法产生类似的美学，还有完整的动画能力。
- **让色块错开步调**：如果每个色块共享一个时长、一起开始，它们会在同一时刻折返，每周期都回到同样的混合。演示给每个色块自己的路径和自己的时长。负的 `animation-delay`（例如 `-10s`）让动画从周期中间开始，所以共享时长的色块不再一起折返，不过整个混合每周期仍然重复。
- **库**：Paper Shaders 有一个 `MeshGradient`，用着色器而不是模糊圆圈画颜色，按速度设置做动画，有纯 JavaScript 版或 React 组件。

## 另见
- [动态渐变背景](../animated-gradient-background/) — 一个慢慢变化的渐变
- [极光](../aurora/) — 高高的彩色光带，而不是圆形色块
- [呼吸光晕](../breathing-glow/) — 一个变大变小的光晕
