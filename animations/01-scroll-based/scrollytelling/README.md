# 滚动叙事

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/scrollytelling/)

## 它是什么
滚动叙事把一幅图片绑在滚动文字讲的故事上。往下读时，文字旁边的图片随你一起变化，在各章之间平滑混合，而不是每章切换一次。演示里你下沉穿过海洋的六层，舷窗里的水变暗，深度数到 10,935 米。

## 什么时候用
- 图表、地图或示意图应随读者推进而演变的数据驱动故事
- 把固定视觉与滚动文字配对的讲解内容（“滚动叙事”新闻）
- 单个主视觉元素随功能描述变化的产品导览
- 任何*点之间*的过渡与点本身同样重要的叙事

## 工作原理
一列固定视觉保持不动，文字章节从它旁边滚过。每帧代码找出哪一章越过了阅读线（舞台 42% 处），再推导出一个分数索引——整数部分是当前章节，小数是向下一章的进度。每个视觉值都是按这个分数做的 `lerp`：

```js
frac = clamp(frac, 0, N - 0.001);
const ci = Math.floor(frac), cf = frac - ci;      // chapter index + fraction
const c = CHS[ci], cn = CHS[Math.min(ci + 1, N - 1)];
const col = lerpCol(c.ac, cn.ac, cf);             // accent color blend
depNum.textContent = Math.round(lerp(c.depth, cn.depth, cf)).toLocaleString();
phBg.style.background = `radial-gradient(circle at 50% 40%, ${lerpCol(c.bg, cn.bg, cf)}, #000 80%)`;
porthole.style.borderColor = col;
```

读取用 `requestAnimationFrame` 门（`ticking`）节流，滚动处理器每帧最多做一次布局工作。因为深度数字和颜色是插值而不是切换，下沉读起来是连续的——0 到 10,935 米流畅地走完，没有跳跃。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 阅读线 | 盒子 42% 处 | 章节顶部越过这条线就成为当前章节；线越高，章节切换越晚，线越低，切换越早 |
| 章节高度 | 至少盒子的 90% | 每章需要多少滚动；章节越高，混合越慢、越细腻 |
| 章节数量 | 6 | 每章都给混合加上它的颜色和深度 |

## 生产注意事项
- **分数，而不是步进**：让滚动叙事感觉平滑的，是章节之间的小数进度。如果只在整数章节变化时切换，你得到的是一张幻灯片——能插值的都插值。
- **固定视觉，滚动文字**：这个模式是在更高的文字列旁边放一个 `position: sticky` 的视觉。这比 JS 固定更便宜、更稳健；让 CSS 把视觉固定在原位。
- **节流读取**：滚动处理器里的 `getBoundingClientRect` 会强制布局。这里的 rAF 门把它限制到每帧一次；视觉更重时，缓存矩形，只在尺寸变化时重新计算。
- **颜色插值**：混合十六进制颜色需要解析到 RGB 通道、再逐个插值——CSS 自己不会在中间值上补间 `background`，所以这一步在 JS 里做。
- **减弱动态效果**：这里没有任何东西自己动；颜色和深度跟随读者的滚动，所以没什么可关的。舷窗渐变上的 CSS 过渡反正也没用：渐变不能过渡，所以混合是在 JavaScript 里算的。
- **等效的库**：Scrollama 是“固定图形 + 滚动步骤”布局的标准原生库；GSAP ScrollTrigger 用 `scrub` 处理插值；Framer Motion 的 `useScroll` + `useTransform` 在 React 里把滚动进度映射到任意动画值。

## 另见
- [吸顶区域](../sticky-section/) — 整个区域固定不动，内容随之变化
- [滚动 scrub 动画](../scrub-animation/) — 滚动让动画正放倒放
- [滚动显现](../reveal-on-scroll/) — 卡片越过一条线时出现
- [阅读进度条](../progress-bar/) — 你读的时候一条横条随之填满
