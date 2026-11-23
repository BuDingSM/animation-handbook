# 进度动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/progress-animation/)

## 它是什么
进度动画通过填充一个形状来显示任务完成了多少：一条横条、一个圆环或一排步骤。和旋转指示不同，它告诉人们还剩多少，这让漫长的等待更容易。在演示里，三种都由同一个百分比驱动。

## 什么时候用
- 有字节进度可用的文件上传和下载
- 多步表单和引导流程（5 步向导，5 步里完成了 3 步）
- 安装和构建过程
- 长文章上的阅读进度指示
- 任何能算出 `loaded / total` 的操作

## 工作原理
**直线横条**：让填充和轨道一样宽，用 `transform: translateX()` 从左边滑进来；轨道的 `overflow: hidden` 藏住还在外面的部分，变换永远不会让页面重新布局。用 CSS 过渡移动它，或用 `requestAnimationFrame` 控制缓动：

```css
.prog-track { overflow: hidden; border-radius: 4px; }

.prog-fill {
  height: 100%;
  width: 100%;
  background: #58a6ff;
  transform: translateX(-100%);   /* empty: the whole fill sits left of the track */
  transition: transform 2000ms ease-out;
}
```

```js
// Or with rAF for custom easing:
function animateProgress(target, duration) {
  const start = performance.now();
  function tick(now) {
    const t = Math.min((now - start) / duration, 1);
    const eased = 1 - (1 - t) ** 3; // ease-out cubic
    fill.style.transform = 'translateX(' + (target * eased - 100) + '%)';
    if (t < 1) requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);
}
```

**圆形圆环**在 SVG 圆上用 `stroke-dashoffset`。半径 `r=32` 的圆，周长是 `2π × 32 ≈ 201px`：

```css
circle {
  fill: none; stroke: #58a6ff; stroke-width: 6;
  stroke-dasharray: 201;
  stroke-dashoffset: 201; /* fully hidden */
  transition: stroke-dashoffset 2s ease-out;
}
```

```js
// p = progress 0..1
circle.style.strokeDashoffset = 201 * (1 - p);
```

**不确定横条**（不知道时长）：改用无限滑动动画：

```css
.indet {
  animation: slide 2s ease-in-out infinite;
  width: 40%;
}
@keyframes slide {
  0%   { transform: translateX(-150%); }
  100% { transform: translateX(350%); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 填充到 | 100% | 一次播放填充多远：50%、75% 或 100% |
| 速度 | 普通 | 填满要花多长时间：慢是 3.2 秒，普通 2 秒，快是 1.2 秒；真实应用里它跟随实际进度 |
| 手感 | 平滑 | 平滑开始快、结尾放慢，感觉自然；轻柔缓入缓出；均匀以一个稳定的节奏填充 |
| 滑动条 | 关 | 用于不知道多长的等待：沿着横条滑动一小段，而不是填充它 |
| 填充颜色 | 蓝色 | 横条、圆环和步骤的颜色 |

## 生产注意事项
- **不要伪造进度**：动画到 90% 再卡住等响应（常见模式）是骗人的，会损害信任。要么动画到一个诚实的检查点，要么用不确定模式。
- **`transition` 对比 `rAF`**：CSS `transition` 更简单，但不能在中途暂停或反向。`rAF` 给你完全的控制——分块上传里进度一阵一阵地来时，这很有必要。
- **无障碍**：把进度条包在 `<progress value="60" max="100">` 里，或加上 `role="progressbar"`、`aria-valuenow`、`aria-valuemin` 和 `aria-valuemax` 属性。
- **GSAP**：`gsap.to(fill, { xPercent: -25, duration: 2, ease: "power2.out" })` 把一个全宽填充滑到 75%。圆形圆环直接动画 `strokeDashoffset`。
- **React**：HTML `<progress>` 元素开箱即用就是无障碍的。Radix UI 的 `<Progress>` 提供无头、有样式的替代品。

## 另见
- [加载旋转指示](../loading-spinner/) — 用于不知道多长等待的旋转指示
- [骨架屏](../skeleton-loader/) — 代替内容的灰色形状
- [对勾绘制](../checkmark-draw/) — 到达终点后的成功标志
- [阅读进度条](../../01-scroll-based/progress-bar/) — 随滚动填充的横条
- [长按确认](../hold-to-confirm/) — 按住时填充的按钮
