# 滚动驱动 3D 旋转

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/scroll-driven-3d-rotation/)

## 它是什么
滚动驱动 3D 旋转把物体的转动绑在滚动位置上。用户滚动经过一个高高的区域时，物体固定在视野里，每滚动一段就进入下一个姿态：转过去、倾斜、变大，然后露出背面。往回滚动会反向播放，就像许多产品页展示新设备那样。

## 什么时候用
- 产品发布页，每个滚动阶段突出一个不同的功能或角度
- 需要互动感又不用视频的硬件或设备落地页
- 3D 物体作为连续文字视觉锚点的叙事区域
- 一步步 3D 演示的新手引导流程

## 工作原理
结合两种手法：**固定**（吸顶定位让 3D 舞台在内容滚动时保持可见）和 **scrub**（滚动进度映射到变换时间线）。

**HTML 结构：**
```html
<div class="scroll-container" style="height: 400vh">
  <div class="sticky-stage">  <!-- position: sticky; top: 0 -->
    <div class="object-3d" id="obj"></div>
  </div>
</div>
```

演示里滚动的盒子就是舞台本身：内容是四个舞台高度（400%），固定的框是一个舞台高度，是内容的四分之一。在整页上同样的结构是一个 400vh 的区域。

**滚动进度 → 变换：**
```js
container.addEventListener('scroll', () => {
  const { scrollTop, scrollHeight, clientHeight } = container;
  const progress = scrollTop / (scrollHeight - clientHeight); // 0..1

  // Map progress to keyframes
  const rx = lerp(keyframes[floor].rx, keyframes[ceil].rx, fraction);
  const ry = lerp(keyframes[floor].ry, keyframes[ceil].ry, fraction);
  const sc = lerp(keyframes[floor].scale, keyframes[ceil].scale, fraction);

  obj.style.transform = `rotateX(${rx}deg) rotateY(${ry}deg) scale(${sc})`;
});

function lerp(a, b, t) { return a + (b - a) * t; }
```

演示的关键帧是四个姿态，滚动的每个四分之一对应一个：向侧面转四分之一圈、向前倾斜、变大（1.4 倍），然后转过去露出背面。

**平滑跟随**每帧加一个 lerp，在原始滚动值和应用的旋转之间缓动——这带来电影感：

```js
const FRAME = 1000 / 60;                   // one frame on a 60 Hz screen, in milliseconds
let last = null;                           // the time of the previous frame; null until the loop has run once
function animate(now) {
  const dt = last === null ? FRAME : Math.min(now - last, 50);  // the first frame counts as one 1/60 s
  const ease = 1 - Math.pow(0.9, dt / FRAME);                   // 0.1 at 60 Hz, 0.19 at 30 Hz
  last = now;
  currentRx += (targetRx - currentRx) * ease;
  currentRy += (targetRy - currentRy) * ease;
  obj.style.transform = `rotateX(${currentRx}deg) rotateY(${currentRy}deg)`;
  requestAnimationFrame(animate);
}
```

立方体每六十分之一秒弥合十分之一的差距，所以一帧要弥合的占比取决于那一帧花了多长时间。立方体在 30、60 或 144 Hz 的屏幕上用同样的时间追上；每帧固定 0.1 的话，在 30 Hz 的手机上要两倍久，在 120 Hz 下只要一半。

演示只在立方体还在追赶时才请求下一帧，落定后就停下。重新开始后的第一帧没有更早的帧可测量，所以算作一个六十分之一秒。「转动幅度」设置把每个姿态的角度乘以 0.6、1 或 1.6，打开减弱动态时演示跳过滑行，直接设置每个姿态。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 转动幅度 | 普通 | 每个姿态让立方体转多远：少是计划角度的 0.6 倍，普通 1 倍，多是 1.6 倍（向侧面转 90°、向前倾 30°，最后向另一边转 180°）；普通时最后一个姿态露出立方体的背面 |
| 平滑跟随 | 开 | 立方体每六十分之一秒弥合与姿态之间十分之一的差距（约半秒弥合 95%），所以它在滚动后滑过去，在任何屏幕上都用同样的时间追上；关掉就精确跟随滚动 |

## 生产注意事项
- **CSS Scroll-Driven Animations API**（Chrome 115+）：`animation-timeline: scroll()` 和 `animation-range` 不用 JavaScript 就能直接驱动 CSS 变换。规范原生覆盖了这一点，但复杂编排的浏览器支持还在跟进。
- **GSAP ScrollTrigger**：生产环境里，`gsap.to(obj, { rotateY: 360, scrollTrigger: { trigger, scrub: 1 } })` 是惯用的实现。`scrub: 1` 增加 1 秒的平滑滞后。
- **按时间，不按帧**：每帧弥合固定占比的缓动会跟着屏幕的刷新率走。像上面那样按自上一帧以来的时间缩放占比，并封顶这个时间（演示用 50ms）。
- **性能**：带 `will-change: transform` 的 3D 元素上的 CSS `transform` 在合成线程上运行——滚动驱动旋转不会触发布局或绘制。
- **手机滚动预算**：手机上 400vh 的滚动意味着用户要滚很多。考虑给手机减小区域高度，或换成滑动驱动（触摸拖动）的交互。

## 另见
- [3D 模型环绕](../3d-model-orbit/) — 时间转动物体，而不是滚动
- [滚动 scrub 动画](../../01-scroll-based/scrub-animation/) — 滚动位置驱动任何动画，不用 3D
- [吸顶区域](../../01-scroll-based/sticky-section/) — 让物体保持在视野里的固定
- [立方体过渡](../../03-page-transitions/cube-transition/) — 点击转动一个页面立方体，而不是滚动
