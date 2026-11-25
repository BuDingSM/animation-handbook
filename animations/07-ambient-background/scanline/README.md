# 扫描线效果

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/scanline/)

## 它是什么
扫描线是老式显像管电视或电脑显示器上光行之间的细深色线条。把它们作为重复条纹图案铺在设计上，就有了 80 或 90 年代复古屏幕的样子。一条柔和的光带慢慢扫过屏幕，增加运动，变暗的角落暗示弯曲的玻璃。

## 什么时候用
- 复古波、赛博朋克或合成波主题的界面
- 游戏 HUD 叠加层和复古街机美学
- “终端”或“命令行”风格的界面
- 有模拟复古感的音乐可视化和媒体播放器界面

## 工作原理
**静态扫描线**——CSS `repeating-linear-gradient` 不用任何 JavaScript 就创建线条图案：

```css
.scanlines {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background: repeating-linear-gradient(
    to bottom,
    transparent       0px,
    transparent       3px,           /* visible row height */
    rgba(0,0,0,0.15)  3px,           /* dark line */
    rgba(0,0,0,0.15)  4px            /* line-gap = 4px total */
  );
}
```

演示的线条间距设置改变这个间隔（`--line-gap`）：`2px` 是密集，`4px` 是标准 CRT 密度，`8px` 更粗、更风格化。

**移动的扫过光带**——一条高高的渐变带，用 `translateY` 做动画：

```css
.beam {
  position: absolute;
  left: 0; right: 0;
  top: -120px;                  /* starts above the visible area */
  height: calc(100% + 120px);   /* 120px taller than the stage */
  background: linear-gradient(
    to bottom,
    transparent                      0%,
    rgba(255, 255, 255, 0.08)       50%,
    transparent                     100%
  );
  background-size: 100% 120px;  /* the band sits at the element's top edge */
  background-repeat: no-repeat;
  animation: sweep 4s linear infinite;
}

@keyframes sweep {
  from { transform: translateY(0); }
  to   { transform: translateY(100%); }   /* 100% = stage height + 120px */
}
```

给 `transform` 而不是 `top` 做动画，让扫过留在合成器上——超大元素纯粹是为了让百分比平移覆盖全部行程，不用 JavaScript 测量舞台。

**CRT 曲面**——径向暗角叠加层让角落变暗，模拟 CRT 屏幕轻微的凸面曲率：

```css
.vignette {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,0,0.5) 100%);
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 线条间距 | 中等 | 线条之间的距离：紧密是 2px，中等 4px，宽松是 8px，每条线 1px 粗；4px 看起来像标准的老屏幕 |
| 线条深浅 | 中等 | 线条有多深：浅是 8% 黑色，中等 15%，深是 30%；更深看起来更复古，但降低对比度 |
| 移动光带 | 开 | 一条柔和的光带一次次扫过屏幕 |
| 光带速度 | 普通 | 光带穿过屏幕需要的时间：慢速 6.4 秒，普通 4 秒，快速 2.4 秒 |
| 光带亮度 | 中等 | 光带有多亮：微弱是 6% 白色，中等 12%，明亮是 24%，沿 120px 光带的中间，顶部和底部淡到没有；应该感觉到的多于看到的 |
| 暗角 | 关 | 柔和的暗角让角落变暗，像老屏幕的弯曲玻璃 |
| 发光文字 | 关 | 字母周围柔和的辉光，像光在老屏幕上扩散 |

## 生产注意事项
- **纯 CSS**：扫描线或光带效果不需要 JavaScript。只有演示的设置用 JavaScript。
- **`pointer-events: none`**：扫描线叠加层不能拦截鼠标事件。叠加元素一定要加 `pointer-events: none`。
- **性能**：`repeating-linear-gradient` 在首次渲染时合成为纹理并缓存。每帧不重绘——基本是免费的。`translateY` 光带动画像在合成器线程上运行。
- **荧光粉辉光**：真实的 CRT 荧光粉发出的光会微微扩散，在明亮文字周围形成柔和的“光晕”。CSS `text-shadow: 0 0 8px currentColor` 近似这个效果。保持微妙——重的辉光会降低易读性。
- **无障碍**：扫描线降低对比度。确保所有文字在应用扫描线叠加层的情况下都符合 WCAG 对比度。用你打算用的深度值测试叠加层。
- **复古波美学**：与[颗粒噪点叠加](../grain-overlay/)（质感）、[色差](../../06-3d-advanced/chromatic-aberration/)（色彩镶边）和深色霓虹调色板搭配得很好。
- **库**：CSS 版本不需要库；在 WebGL 场景上，three.js 的后期处理库有一个带密度和滚动速度设置的 `ScanlineEffect`。

## 另见
- [颗粒噪点叠加](../grain-overlay/) — 细腻颗粒闪烁的质感
- [漏光](../light-leak/) — 暖光漫入，像胶片相机的缺陷
- [色差](../../06-3d-advanced/chromatic-aberration/) — 颜色在边缘分裂，像坏掉的信号
