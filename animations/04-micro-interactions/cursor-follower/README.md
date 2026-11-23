# 光标跟随

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/cursor-follower/)

## 它是什么
光标跟随者是一个小图形，带着一点延迟跟在鼠标指针后面，滑行着追赶、赶上时慢慢减速，而不是粘在指针上。它反转下方任何东西的颜色，在任何背景上都看得见——深色区域上是浅色，浅色区域上是深色，不用为每种背景选颜色。

## 什么时候用
- 作品集和代理类网站，光标本身成为品牌体验的一部分
- 互动讲故事体验，光标反应能加强沉浸感
- 着陆页，悬停目标会让跟随者变大，给出触感反馈
- 任何默认操作系统光标太不显眼、不符合预期美感的网站

## 工作原理
跟踪 `pointermove` 来记录目标坐标（`mx`、`my`）。每一帧动画，用线性插值（lerp）把跟随者的位置插向目标。要缩小的差距份额按自上一帧以来的时间缩放，这样跟随者在 30、60 或 120 Hz 的屏幕上保持同样的速度：

```js
let mx = 0, my = 0;       // pointer position
let fx = 0, fy = 0;       // follower position
let last = null;          // time of the previous frame
const FRAME = 1000 / 60;  // one frame of a 60 Hz screen, in ms
const ease = 0.15;        // fraction of the gap to close in one 60 Hz frame

document.addEventListener('pointermove', e => { mx = e.clientX; my = e.clientY; });

function loop(ts) {
  const dt = last === null ? FRAME : Math.min(ts - last, 50);   // ms since the last frame, at most 50
  last = ts;
  const a = 1 - Math.pow(1 - ease, dt / FRAME);                  // equals ease at 60 Hz
  fx += (mx - fx) * a;
  fy += (my - fy) * a;
  follower.style.transform = `translate(${fx}px, ${fy}px)`;   // transform only: nothing is laid out again
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```

每个 60 Hz 帧会缩小跟随者和光标之间剩余差距的 15%。长一倍的帧缩小 28%，短一半的帧缩小约 8%，所以跟随者在任何屏幕上大约 18 帧的时间（约 300ms）内就能走完约 95% 的路，产生平滑的延迟，永远不会完全追上。第一帧算作一个 60 Hz 帧，超过 50ms 的间隔（标签从后台回来）按 50ms 算。

`mix-blend-mode: difference` 这个 CSS 属性让跟随者反转它下方任何背景颜色：

```css
#follower {
  position: fixed;
  left: 0; top: 0;                 /* moved only by transform */
  mix-blend-mode: difference;
  pointer-events: none;
  will-change: transform;
  z-index: 9999;
}
#follower .shape {
  width: 32px; height: 32px;
  border-radius: 50%;
  background: #fff;
  transform: translate(-50%, -50%);
  transition: transform .2s ease;
}
#follower.expanded .shape { transform: translate(-50%, -50%) scale(2.5); }
```

悬停反应时，在交互目标上把跟随者放大：

```js
stage.addEventListener('pointermove', e => {
  follower.classList.toggle('expanded', !!e.target.closest('a, button, [data-hover]'));
});
```

键盘也能驱动它：Tab 聚焦舞台上的一层透明层，把一个看不见的指针放在它的中间（或按键最后离开的地方），方向键一次移动一步（舞台较短边的 6%，按住 Shift 一次三步），走的和鼠标同一套代码，所以圆点会滑行着追赶，并在按钮上变大。Esc，或按 Tab 离开舞台，会像指针离开一样隐藏圆点。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 延迟 | 中等 | 圆点跟随得有多紧：在 60 Hz 屏幕上，每帧缩小与指针差距的 35%（短）、15%（中等）或 6%（长），其他屏幕按帧时间缩小相应的份额，所以圆点在任何地方大约 0.1、0.3 或 0.8 秒内走完 95% 的路；延迟越长感觉越沉 |
| 大小 | 中等 | 圆点直径 20px、32px 或 48px |
| 反转下方颜色 | 开 | 圆点反转下方的颜色，所以在浅色和深色区域都看得见；关时，它是一个半透明的橙色圆点 |
| 形状 | 圆形 | 圆形、圆环或方形 |
| 悬停按钮时变大 | 开 | 指针在按钮上时，圆点长到两倍半大小 |

## 生产注意事项
- **触屏设备**：光标效果不适用于触摸。检测 `(pointer: coarse)` 并完全隐藏跟随者——绝不要在触摸上模拟光标。演示把圆点送到每次轻点的位置，只是为了让效果在手机上也能看到。
- **性能**：`position: fixed` 加 `will-change: transform` 会把跟随者提升到自己的 GPU 层。演示用 `transform: translate(x, y)` 移动它，用 `scale()` 放大它，所以移动时什么都不用重新布局。
- **延迟校准**：`ease` 是 60 Hz 一帧的份额，演示按帧时间缩放它，所以 `ease = 0.15` 在 30、60 或 120 Hz 屏幕上都给出约 300ms 的延迟。普通的每帧插值在 120 Hz 显示器上约 150ms，在 30 Hz 手机上约 600ms，所以跟随者在不同设备上感觉时快时慢。
- **隐藏光标**：跟随者激活时，给 `body` 加 `cursor: none` 来隐藏操作系统光标。触摸时，或光标离开窗口时，再恢复。
- **GSAP QuickTo**：`const xTo = gsap.quickTo(follower, 'x', { duration: 0.3 })` 是 GSAP 优化过的光标跟随者模式；它用秒数表示持续时间，所以是基于时间的，不用算 `dt`。
- **`@motionone/animate`**：GSAP 的轻量替代品，适合光标跟随者；没有摇树优化的开销。

## 另见
- [悬停状态动画](../hover-state/) — 指针在上面时有反应的项目
- [提示气泡浮现](../tooltip-reveal/) — 指针停留处出现的标签
- [点击涟漪](../click-ripple/) — 从点击的确切位置扩散的涟漪
- [图片拖尾](../image-trail/) — 沿着指针路径弹出又淡出的图片
