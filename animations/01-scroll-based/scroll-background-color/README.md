# 滚动驱动背景色

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-background-color/)

## 它是什么

滚动驱动背景色给页面的每节自己的颜色，滚动时从一个混合到下一个，整个页面好像换了心情。在两节中间，背景是两种颜色的混合。背景变亮时，文字切换成深色，保持可读。

## 什么时候用

- 每个案例研究或区域都有品牌色的作品集和代理机构网站
- 想要旅程感、又不想用重型图片的长篇落地页
- 情绪变化（由暗到亮、由冷到暖）强化叙事的故事页面
- 作为全幅区域图片的廉价替代——颜色变化读起来就像场景变化

## 工作原理

滚动进度归一化到 `0..1`，缩放到调色板的分段里，相邻的两种颜色做插值。平滑步进（smoothstep）缓和每次过渡，让混合先加速再减速，而不是线性移动：

```js
const p   = clamp(scrollTop / maxScroll, 0, 1);
const seg = p * (palette.length - 1);
const i   = Math.min(Math.floor(seg), palette.length - 2);
let   t   = seg - i;

t = clamp((t - (1 - window) / 2) / window, 0, 1); // blend window remap
t = t * t * (3 - 2 * t);                          // smoothstep

const r = Math.round(a[0] + (b[0] - a[0]) * t);   // per-channel lerp
```

对比度翻转计算插值颜色的相对亮度，越过阈值时切换一个类，把整节文字从白色换成黑色：

```js
const lin = c => (c /= 255) <= 0.03928 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4;
const lum = 0.2126 * lin(r) + 0.7152 * lin(g) + 0.0722 * lin(b);   // relative luminance
stage.classList.toggle('light', lum > 0.18);   // white and black text contrast equally at 0.18
```

滚动处理器用 rAF 合并，不做布局读取；`maxScroll` 在初始化、字体加载完和窗口每次调整大小时缓存。背景写入有保护——通道值先取整，生成的字符串没变就跳过写入。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 颜色 | 黄昏 | 调色板，每节一个颜色：黄昏保持深色；日光经过浅色，森林经过浅绿，文字在这些颜色上变深 |
| 混合 | 渐变 | 渐变在两节之间全程改变颜色；快速保持每种颜色，只在边界附近、从一节到下一节滚动的中段 30% 里改变；硬切换在边界处直接切换、完全不混合，用来说明混合为什么重要 |

## 生产注意事项

- `background-color` 只绘制、不触发布局，所以每帧写一次很便宜。但写入还是要有保护：取整后的颜色字符串没变就跳过，滚动处理器里永远不要读布局。
- 这里为了清晰展示的是普通 RGB 插值，但它会经过颜色立方体的灰色中心——饱和色之间的混合可能变浑浊。生产中要在感知均匀的空间里插值：`oklch()` 或 `color-mix(in oklch, ...)` 能让中间色保持鲜艳。
- 文字对比度必须计算，不能想当然。WCAG 要求正文在*整个*混合过程中大约达到 4.5:1，不只是调色板的端点——两个通过的颜色之间的过渡可能经过一个不达标的中间色。检查最坏情况的中间色，或者选留有充足余量的文字颜色组合。
- 只用 CSS 也可以实现，用 `animation-timeline: scroll()` 驱动一个 `background-color` 的关键帧动画；加 `steps()` 可以复现跳变版本。支持还不普遍，所以把它当作 JS 做法之上的渐进增强。
- 如果网站有深色模式，调色板需要深色模式的对应版本，亮度阈值也可能要调整——深色模式里的“浅色”背景比浅色模式里的同一种颜色更刺眼。
- 颜色变化不是前庭运动，所以这个演示在 `prefers-reduced-motion: reduce` 下特意让混合继续运行。这个偏好针对的是运动——视差、缩放、倾斜——而不是绘制。叠加在上面的、基于变换的装饰还是应该禁用。
- GSAP 的 ScrollTrigger 用 `scrub: true` 把一个 `backgroundColor` 补间绑定到滚动位置，可以替代这里的混合代码；为每次混合挑选文字颜色还是要自己写。

## 另见

- [滚动叙事](../scrollytelling/) — 故事滚动经过时，文字旁边的图片随之变化
- [动态渐变背景](../../07-ambient-background/animated-gradient-background/) — 颜色自己变化，不需要滚动
- [区域擦除切换](../section-wipe/) — 每节从上一节上面滑上来
