# SVG 路径动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/svg-path-animation/)

## 它是什么
SVG 路径动画让线条画看起来像手画出来的一样。每条线藏在一段和线一样长的虚线后面，把虚线滑走，就从一端到另一端露出线条。整幅画从一开始就在页面里，带有标签，所以屏幕阅读器总能找到它。

## 什么时候用
- 页面加载时“画出来”的插画图标或 Logo 揭示
- 品牌文案的签名式手写效果
- 随用户进度填满的步骤完成指示
- 在内容周围自己画出来的装饰边框和画框

## 工作原理
每条 SVG 路径都有一个可通过 `path.getTotalLength()` 获得的总描边长度。把 `stroke-dasharray` 设为那个长度，就得到一段与整条路径一样长的虚线。把 `stroke-dashoffset` 设为同样的长度会隐藏它（间隙从位置 0 开始，盖住整个描边）。把偏移从全长动画到 0，就露出描边：

```js
const path = document.querySelector('.draw-path');
const length = path.getTotalLength();

// Set up hidden state
path.style.strokeDasharray  = length;
path.style.strokeDashoffset = length;    // fully hidden
path.style.transition = 'none';

// Make the browser apply the hidden state, then animate
path.getBoundingClientRect();
path.style.transition = 'stroke-dashoffset 1500ms ease-out';
path.style.strokeDashoffset = 0;         // fully revealed
```

多条路径按顺序画时，演示让每条线在前一条之后 `gap` 开始：绘制时间 `ms` 的 30%，所以一条线在前一条还没画完时就开始了。`ms` 是「速度」设置（普通 1500ms，慢动作里三倍长），`ease` 是「手感」设置，`rev` 是「从另一端开始」开关；`later` 是还会保存定时器 id 的 `setTimeout`，这样「重播」可以取消还在等待的：

```js
const paths = buildPaths(), ms = speed(), gap = ms * 0.3, rev = revTog.checked;
paths.forEach(path => {
  const len = path.getTotalLength();
  path.style.strokeDasharray  = len;
  path.style.strokeDashoffset = rev ? -len : len;   // hidden either way
  path.style.transition = 'none';
});
paths.forEach((path, i) => later(() => {
  path.getBoundingClientRect();       // apply the hidden state first
  path.style.transition = `stroke-dashoffset ${ms}ms ${ease}`;
  path.style.strokeDashoffset = 0;
}, 20 + i * gap));
```

**最后填充**——最后一条线画完后（`20 + (paths.length - 1) * gap + ms + 50` 毫秒时），一个 class 把填充淡入。路径必须以 `fill: transparent` 开始，而不是 `fill: none`：填充只能从一种颜色淡入，从 `none` 会一下子跳出来。

```css
.draw-path { fill: transparent; fill-opacity: .15; stroke: var(--stroke-color); }
.draw-path.fill-done { fill: var(--stroke-color); transition: fill var(--fill-dur) ease; }
```

```js
// clearing the inline transition lets the class's fill transition apply
if (fillTog.checked) paths.forEach(path => { path.style.transition = ''; path.classList.add('fill-done'); });
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 画什么 | 图标 | 画什么：带星星的房子、流畅的签名，或圆形标志 |
| 速度 | 普通 | 每条线画出来要多长时间：慢是 2400ms，普通 1500ms，快是 900ms；下一条线在 30% 的时间后开始 |
| 线条粗细 | 中等 | 绘图在完整 400px 宽时，稀薄是 1px，中等 2px，浓厚 4px，缩小以适应更小的盒子时会变细；细线显得精确，粗线显得醒目 |
| 手感 | 平滑 | 平滑在结尾放慢，像手收笔一样；轻柔两头缓动；均匀保持一个速度 |
| 线条颜色 | 蓝色 | 线条的颜色，也是填充的颜色 |
| 从另一端开始 | 关 | 每条线从末端画回起点 |
| 最后填充 | 关 | 线画完后，给形状填上线条颜色的淡色 |

## 生产注意事项
- **必须用 `getTotalLength()`**：硬编码 `stroke-dasharray` 在路径变化时会失效。始终在运行时测量。异步加载的 SVG，在元素加入 DOM 后再测量。
- **路径方向**：描边从路径的起点（第一个 `M` 命令）开始画。要控制哪一端先画，反转路径数据，或从*负*偏移开始（`-length` → `0`），它以同样的方式隐藏描边，但从远端揭开。`0` → `length` 的动画是另一种效果：它擦掉已经画好的路径。
- **`stroke-dasharray` 简写**：`stroke-dasharray: length length`（重复）和 `stroke-dasharray: length` 等价——单个值把虚线和间隙都设为同样的长度。
- **GSAP DrawSVG 插件**：自动处理 `getTotalLength()`、偏移计算和动画排序。复杂的多路径插画必备。`gsap.from(path, { drawSVG: 0 })`。
- **Framer Motion**：`<motion.path initial={{ pathLength: 0 }} animate={{ pathLength: 1 }} />`——`pathLength` 是 0–1 的简写，内部管理 `dasharray`/`dashoffset`。

## 另见
- [对勾绘制](../../04-micro-interactions/checkmark-draw/) — 同样的手法，画一个成功对勾
- [裁剪揭示文字](../../05-text-typography/text-clip-path-reveal/) — 一行行揭开的文字
- [描边填充](../../05-text-typography/outline-to-fill/) — 空心字填满颜色
- [手写描绘](../../05-text-typography/handwriting-draw/) — 同样的手法一笔一画写出一个词
