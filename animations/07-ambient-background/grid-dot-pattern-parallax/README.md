# 网格 / 圆点图案视差

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/grid-dot-pattern-parallax/)

## 它是什么
网格或圆点图案视差在 Linear 和 Vercel 等开发者工具网站上很常见，它在内容后面放一个淡淡的重复圆点或线条图案，并让它朝指针的反方向微微移动。移动很小，只有指针移动的几个百分点，所以图案看起来比内容深一点：深度更多是感觉到的，而不是看到的。没有指针在上面时，网格自己慢慢漂移。

## 什么时候用
- 技术和开发者工具落地页，网格图案传达“结构化、精确”
- 任何主视觉区，纯深色背景太平、运动又太分散注意力
- 仪表盘模板，网格图案微妙地呼应产品数据网格的美学
- 作品集主视觉区，图案提供质感又不与内容抢注意力

## 工作原理
图案用 CSS `background-image` 加平铺的径向或线性渐变创建。图案层比容器稍大（每边溢出 10%），留出漂移空间。跟踪鼠标位置，映射为平移偏移量：

```js
stage.addEventListener('pointermove', e => {
  const rect = stage.getBoundingClientRect();
  const normX = (e.clientX - rect.left) / rect.width  - 0.5; // -0.5 to 0.5
  const normY = (e.clientY - rect.top)  / rect.height - 0.5;

  const STRENGTH = 0.05; // 5% of stage width/height
  const dx = -normX * STRENGTH * rect.width;
  const dy = -normY * STRENGTH * rect.height;

  patternLayer.style.transform = `translate(${dx}px, ${dy}px)`;
});
```

指针事件覆盖鼠标、触控笔和手指。演示舞台有 `touch-action: pan-y`，所以横向拖动操纵网格，竖向滑动仍然滚动页面，只有鼠标会打开光标圆环和光。

**CSS 圆点网格**，用径向渐变：

```css
.dot-grid {
  background-image: radial-gradient(
    circle,
    rgba(88, 166, 255, 0.25) 1.5px,   /* dot size */
    transparent 0
  );
  background-size: 28px 28px;  /* dot spacing */
}
```

**CSS 线条网格**，用交叉的线性渐变：

```css
.line-grid {
  background-image:
    linear-gradient(rgba(88, 166, 255, 0.25) 1px, transparent 1px),
    linear-gradient(90deg, rgba(88, 166, 255, 0.25) 1px, transparent 1px);
  background-size: 28px 28px;
}
```

**自动漂移**，没有指针在舞台上时（触屏上两次拖动之间也是）：

```js
let clock = 0, last = null;
function autoLoop(now) {
  clock += last === null ? 0 : Math.min(now - last, 50);   // the drift's own clock stops while paused
  last = now;
  const dx = -Math.sin(clock * 0.0002)  * STRENGTH * W * 0.4;
  const dy = -Math.cos(clock * 0.00014) * STRENGTH * H * 0.3;
  patternLayer.style.transform = `translate(${dx}px, ${dy}px)`;
  requestAnimationFrame(autoLoop);
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 图案 | 圆点 | 圆点网格、线条网格，或两者都有 |
| 强度 | 中等 | 网格移动多远：微妙是指针移动的 3%，中等 5%，强烈 10%，没有指针时的漂移也按同样比例（中等时最多是舞台宽度的 2% 和高度的 1.5%）；不到 10% 时感觉到的多于看到的 |
| 间距 | 中等 | 圆点或线条之间的距离：紧密是 20px，中等 28px，宽松是 44px |
| 图案颜色 | 蓝色 | 圆点和线条的颜色，以 25% 显示，保持淡淡的 |
| 第二层 | 关 | 更稀疏的大圆点层移动一半距离，在两层之间增加真实的深度 |
| 指针下的光 | 开 | 一个淡淡的圆形光晕跟随指针 |

## 生产注意事项
- **`background-size` 控制间距，不是圆点大小**：圆点大小是渐变断点值（例如 `1.5px`）。`background-size` 是平铺重复的间隔。
- **触屏设备没有鼠标**：把 `auto-drift` 变体（缓慢的正弦漂移）作为触摸后备方案。用 `(pointer: coarse)` 媒体查询检测。
- **图案层上的 `will-change: transform`**：层很大、平移频繁时（每次 `mousemove` 事件）有用。在 `mouseenter`/`mouseleave` 时动态添加/移除。
- **用 `transform: translate()`，而不是 `background-position`**：`translate` 用合成器线程；`background-position` 触发绘制。视差偏移永远用 `transform`。
- **SVG 图案替代方案**：`<svg><pattern>` 可以通过 `background-image: url("data:image/svg+xml,...")` 把更复杂的重复图案（六边形、三角形等）定义为背景图。这样图案保持纯 CSS/SVG，没有画布开销。
- **库**：Parallax.js 按每层的 `data-depth` 让各层相对指针移动，有陀螺仪时相对手机倾斜移动；用 GSAP 时，`quickTo()` 让图案缓动到每个新的指针位置。

## 另见
- [漂浮元素](../floating-elements/) — 自己漂移的图形，而不是网格
- [2.5D / 伪 3D](../../06-3d-advanced/2-5d-pseudo-3d/) — 许多层在不同深度移动
- [视差 3D 倾斜](../../06-3d-advanced/parallax-3d-tilt/) — 朝指针倾斜的卡片
