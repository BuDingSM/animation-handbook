# 动态毛玻璃

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/glassmorphism-animated/)

## 它是什么
动态毛玻璃把卡片做成磨砂玻璃：半透明的着色、身后东西的模糊，和一条细细的亮边。只有玻璃后面有彩色的东西在动时它才成立，所以演示在三张卡片后面漂着柔和的彩色色块：一张是静止的磨砂，一张的模糊会呼吸，一张的着色会慢慢变化。

## 什么时候用
- SaaS 仪表盘和产品落地页，玻璃卡片展示指标或功能
- 盖在底层内容模糊版之上的弹窗
- 带模糊背景的导航叠加层和侧边栏
- 任何深色主题界面，层与层之间的深度需要传达出来，又不想用粗边框

## 工作原理
三个 CSS 属性做出玻璃效果：

```css
.glass-card {
  background: rgba(88, 166, 255, 0.12);   /* semi-transparent tint */
  backdrop-filter: blur(12px) saturate(1.8);
  -webkit-backdrop-filter: blur(12px) saturate(1.8);
  border: 1px solid rgba(255, 255, 255, 0.18);   /* light edge highlight */
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3),
              inset 0 1px 0 rgba(255, 255, 255, 0.2); /* top edge catch-light */
}
```

**动态模糊**用 `@keyframes` 循环改变 `backdrop-filter` 的值：

```css
@keyframes blur-breath {
  0%, 100% { backdrop-filter: blur(8px) saturate(1.8); }
  50%       { backdrop-filter: blur(20px) saturate(2); }
}
```

**动态背景色块**——纯 CSS 关键帧运动，让玻璃后面有值得模糊的东西：

```css
.blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  animation: blob-move 8s ease-in-out infinite alternate;
}

@keyframes blob-move {
  0%   { transform: translate(0, 0)      scale(1); }
  33%  { transform: translate(40px,-30px) scale(1.1); }
  66%  { transform: translate(-20px, 40px) scale(0.95); }
  100% { transform: translate(30px, 10px) scale(1.05); }
}
```

**玻璃后面的图案**让磨砂看得见。本来就柔和模糊的颜色留给玻璃的东西不多，所以「轻」「中等」「重」三档磨砂看起来会差不多。演示在颜色和卡片之间放了一层淡淡的白点图案。每张卡片模糊身后的圆点：「关」让它们保持清晰，「轻」让它们变柔和，「重」几乎把它们抹掉。图案向舞台边缘渐渐淡出，所以舞台的大部分仍然是柔和的颜色：

```css
.stage::after {
  content: "";
  position: absolute;
  inset: 0;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.16) 0 16px, transparent 18px) 0 0 / 56px 56px;
  -webkit-mask-image: radial-gradient(ellipse closest-side, #000 68%, transparent);
  mask-image: radial-gradient(ellipse closest-side, #000 68%, transparent);
}
```

**文字下面的一层深色**让文字保持可读。明亮的颜色从玻璃后面经过，白字压在上面时，对比度可能远低于小文字需要的 4.5:1。每张卡片都有一层半透明的深色，位于着色和模糊背景之上、文字之下（卡片的 `z-index` 给这层的 `z-index: -1` 在卡片内留了位置）：

```css
.glass-card::before {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: rgba(6, 8, 14, 0.54);
  z-index: -1;
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 磨砂程度 | 中等 | 玻璃把身后的东西模糊到什么程度：轻是 6px，中等 12px，重是 20px；关是不模糊卡片，用来对比 |
| 着色强度 | 中等 | 玻璃加了多少颜色：淡淡是以 6% 的强度填充着色，中等 12%，强烈 25% |
| 速度 | 普通 | 身后的颜色漂完一条路径要多长时间：慢是 13 秒，普通 8 秒，快是 5 秒 |
| 玻璃颜色 | 蓝色 | 玻璃的颜色 |
| 呼吸磨砂 | 开 | 中间卡片的模糊每 4 秒从 8px 变大到 20px 再回来 |
| 变色 | 开 | 右边卡片的着色每 8 秒依次经过蓝色、绿色、紫色和橙色 |

## 生产注意事项
- **`backdrop-filter` 很耗 GPU**：浏览器每帧都要把元素身后的所有东西截一张快照再模糊。大面积玻璃或低端硬件上会掉帧。玻璃卡片保持小巧；避免全屏玻璃叠加层。
- **仍需要 `-webkit-` 前缀**：Safari 即使在 2024 年也需要 `-webkit-backdrop-filter`。带前缀和不带前缀的属性都要写。
- **背景必须有内容**：`backdrop-filter` 模糊的是元素身后的东西。如果背景是纯色，模糊什么也不做——玻璃看起来像脏塑料。玻璃后面需要色彩丰富、对比强的内容，效果才看得见。
- **磨砂需要身后的细节来显现**：柔和颜色的模糊随程度增长变化很小，所以「磨砂程度」设置看起来可能像没起作用。在玻璃后面放带边缘的东西，比如图案、形状或图片，演示就是用圆点这样做的。
- **保持文字可读**：在动画的几个时刻，测量文字相对真正身后内容的对比度，而不是相对着色。演示用一层半透明的深色把玻璃在文字下面压暗，所以每段文字即使压在最亮的颜色上也保持 4.5:1 或更好。
- **深色模式**：毛玻璃需要足够深的背景，读起来才是玻璃而不是白雾。在深色主题上效果更好。
- **改动运行中动画的长度**：新的 `animation-duration` 会把运行中的 CSS 动画放到周期的另一个点上，于是它会跳一下。演示把负延迟写成时长的分数，并把每个色块的 `currentTime` 乘以新长度除以旧长度，所以改「速度」时每个色块都留在原地。
- **与关掉模糊对比**：演示的「磨砂程度」设置里有「关」这一档。如果关掉模糊后页面明显更快，就要重新考虑这个设计。
- **不需要库**：`backdrop-filter` 是纯 CSS，玻璃后面漂移的色块也是纯 CSS；只有玻璃要和页面的其余部分同步移动时，Framer Motion 或 GSAP 才帮得上忙。

## 另见
- [网格渐变](../../07-ambient-background/mesh-gradient/) — 作为背景漂移的柔和彩色色块
- [WebGL 着色器动画](../webgl-shader-animation/) — 在显卡上画出的流动颜色
- [弹窗展开](../../04-micro-interactions/modal-expand/) — 弹窗，磨砂玻璃常出现的地方
- [液态玻璃](../liquid-glass/) — 透明玻璃，在边缘弯曲身后的东西而不是模糊它
