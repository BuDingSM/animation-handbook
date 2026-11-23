# 上滑揭示

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/slide-up-reveal/)

## 它是什么

上滑揭示让一行文字从一条看不见的边缘后方升入视野，就像从一个表面下面升上来一样。文字在动，但线以下的东西一直藏着，所以只看得到它的到达。这是标题和主视觉文案的经典入场。

## 什么时候用
- 落地页和案例研究里的主视觉和标题文案
- 逐行揭示，每行都带一点交错升起
- 滚动进入视野时动起来的章节标题
- 任何单纯淡入淡出显得太平、想让文字感觉是被*托*到位的地方

## 工作原理
有两种办法做出完全一样的视觉效果，「揭示方式」设置可以在它们之间切换（「滑动」是方法 A，「擦除」是方法 B）。**方法 A**把每行包在一个 `overflow: hidden` 容器里，把里面的文字用 `translateY(110%)` 推下去；加上 `.in` 就回到 `0`。**方法 B**让文字留在原地，给 `clip-path: inset()` 做动画，从底部完全裁剪到完全打开——不需要包装元素：

```css
/* Method A — translateY inside a clipping wrapper */
.line-wrap { overflow: hidden; }
[data-method="translatey"] .line { transform: translateY(110%); }
[data-method="translatey"] .line.in { transform: translateY(0); }

/* Method B — clip-path, no wrapper required */
[data-method="clippath"] .line { clip-path: inset(100% 0 0 0); }
[data-method="clippath"] .line.in { clip-path: inset(0% 0 0 0); }
```

多行通过每行的 `--d` 序号乘以 `transition-delay` 里的 `--stagger` 变量来交错：

```css
.line { transition-delay: calc(var(--d, 0) * var(--stagger)); }
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 每行升起来要多长时间：「慢」是 1100ms，「普通」700ms，「快」400ms |
| 行间延迟 | 中等 | 下一行开始前的等待：「短」是 30ms，「中等」60ms，「长」120ms；40 到 80ms 读起来像级联 |
| 揭示方式 | 滑动 | 两种做法看起来几乎一样：在隐藏的边缘后面把文字向上滑，或者在原地把它擦出来 |
| 感觉 | 平滑 | 「平滑」慢慢停下；「弹回」加一点点落定；「均匀」保持一个稳定的速度 |

## 生产注意事项
- **用 `translateY(110%)`，不是 `100%`。**多出来的 10% 盖住下行字母（g、y、p）和行高余量，移动之前不会有一小片字形从边界上方露出来。
- **按取舍选方法。**包装元素里的 `translateY` 合成起来稍便宜、动画更平滑，但需要多一个 `.line-wrap` 元素。`clip-path` 不需要包装元素，文字保持为一个可选中的块，绘制成本稍高一点。对用户来说，两者看起来一样。
- **减弱动态：**在 `prefers-reduced-motion` 下，演示完全去掉 transform 和裁剪，退回到单纯的不透明度淡入淡出，什么都不滑动。
- **库里的等价写法：**GSAP 的 SplitText 插件加一个 `y: '110%'` 补间是经典的生产配方；Framer Motion 在 `overflow-hidden` 包装元素里给 `y` 做动画，配合 `staggerChildren`；Motion One 的 `animate` 用一个从序号得出的 `delay`。

## 另见
- [拆字揭示](../split-text-reveal/) — 文字一块一块出现，而不是一行一行
- [幕布揭示](../curtain-reveal/) — 一块彩色面板盖住它，然后滑走
- [剪裁路径揭示](../clip-path-reveal/) — 一个形状揭开任何元素
- [滑入](../slide-in/) — 从任何一边滑进来，什么都不遮挡
