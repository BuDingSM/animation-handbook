# 动态渐变背景

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/animated-gradient-background/)

## 它是什么
动态渐变背景在内容后面慢慢变换颜色，周期很长（通常 15 到 30 秒），所以页面感觉有生气，却没有什么会抓住眼球。渐变画得比它填充的区域大得多，慢慢来回滑动，或者它的颜色沿着色轮转动。这是最轻量的氛围效果：浏览器完全不用脚本就能让它动起来。

## 什么时候用
- 需要微妙生气、又不分散标题文案注意力的主视觉区
- 品牌调色板应该感觉动态的 SaaS 落地页
- 需要温暖感、又不用照片的深色模式仪表盘
- 静态背景感觉太平、视频又太重的任何场景

## 工作原理
**滑动**（位置移动）——最常见的变体。渐变比元素大得多（`background-size: 400% 400%`），`background-position` 在一个慢循环里做动画：

```css
.bg {
  background: linear-gradient(135deg, #0d1a2e, #1a0d2e, #0d2e1a, #2e1a0d, #0d1a2e);
  background-size: 400% 400%;
  animation: grad-shift 20s ease infinite;
}

@keyframes grad-shift {
  0%   { background-position: 0%   50%; }
  50%  { background-position: 100% 50%; }
  100% { background-position: 0%   50%; }
}
```

**色轮**（色相旋转）——给静态渐变加一个旋转的滤镜：

```css
.bg {
  background: linear-gradient(135deg, #58a6ff, #d2a8ff, #56d364);
  animation: hue-spin 20s linear infinite;
}
@keyframes hue-spin { to { filter: hue-rotate(360deg); } }
```

注意：渐变元素上的 `filter` 也会影响所有子元素——把内容包在单独的 `z-index` 层里，或者把滤镜加到伪元素上。

**亮色带**在两端深色、中间颜色更亮的渐变上运行同样的滑动动画（`background-size: 300% 300%`），周期是一倍半。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 渐变方式 | 滑动 | 滑动让一个大而柔和的渐变（盒子宽高的四倍）来回移动；色轮让每种颜色沿着色轮转动；亮色带让两端深色之间的亮色滑动（盒子宽高的三倍），慢一倍半 |
| 速度 | 普通 | 一个周期需要的时间：慢速 32 秒，普通 20 秒，快速 12 秒；不到 8 秒就会分散注意力 |
| 颜色 | 冷色 | 渐变流转经过的颜色：冷蓝色和紫色、日落橙色、海洋蓝和绿色，或灰色 |
| 柔和色彩 | 关 | 去掉大部分色彩（30% 饱和度），背景更安静 |

## 生产注意事项
- **纯 CSS——不需要 JavaScript**：整个效果就是一个 CSS 动画，没有 `requestAnimationFrame`，也没有画布。
- **每帧都会重绘**：`background-position` 不是合成器能自己做动画的属性，所以浏览器每帧都会重绘渐变（不过没有布局）。在整个主视觉区这通常没问题；最省的版本是把渐变画在一个超大的层上，用 `transform: translate()` 滑动它，合成器运行它时不用重绘。
- **无缝循环**：在渐变末尾重复第一个颜色断点，确保回到起点的过渡平滑。没有它，动画重启动时会有明显的“跳变”。
- **用 `@property` 实现平滑的色相断点动画**：原生 CSS 自定义属性插值（`@property` 加 `syntax: '<color>'`）可以在渐变内给单个颜色断点做动画——这是不需要 `background-size` 技巧的新方法。
- **性能**：在低端设备上，大面积重绘的渐变会掉帧；`will-change: background-position` 阻止不了重绘，`transform` 版本可以。
- **库**：Granim.js 在 `<canvas>` 上画渐变，并在命名的渐变状态之间移动，适合 CSS 循环不够用的时候。

## 另见
- [网格渐变](../mesh-gradient/) — 柔和的彩色色块漂移、交融
- [呼吸光晕](../breathing-glow/) — 一个柔和的光晕变大变小
- [极光](../aurora/) — 彩色光带如北极光般摇曳
