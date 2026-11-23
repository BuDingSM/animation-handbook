# 按钮按压缩放

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/button-press-scale/)

## 它是什么
按钮按压缩放让按钮在按住时微微缩小，松开时弹回来。屏幕上没有真正的按钮行程，所以这个小小的缩小代替了按实体按钮的手感，并确认按压已经登记。

## 什么时候用
- 主要操作按钮（提交、确认、支付、发送）
- 任何漏点会让人沮丧的按钮
- 以触摸目标代替鼠标点击的移动优先界面
- 需要比涟漪更强的按压确认的图标按钮

## 工作原理
按压和松开用两个不同的过渡处理，时间也不一样——按压快，松开慢一些，还可以选择带回弹：

```css
:root {
  --press-scale: 0.95;
  --press-dur: 80ms;
  --release-dur: 180ms;
  --release-ease: cubic-bezier(.34, 1.56, .64, 1);
}

.btn {
  transition: transform var(--release-dur) var(--release-ease);
  transform-origin: center;
}
.btn.pressed {
  transition: transform var(--press-dur) ease;
  transform: scale(var(--press-scale));
}
```

```js
const press = () => btn.classList.add('pressed');
const release = () => btn.classList.remove('pressed');

btn.addEventListener('pointerdown', e => { press(); btn.setPointerCapture(e.pointerId); });
btn.addEventListener('pointerup', release);
btn.addEventListener('pointercancel', () => btn.classList.remove('pressed'));

// Keyboard support
btn.addEventListener('keydown', e => { if (e.key === ' ' || e.key === 'Enter') press() });
btn.addEventListener('keyup',   e => { if (e.key === ' ' || e.key === 'Enter') release() });
btn.addEventListener('blur', release);
```

不对称的时间是关键：按压（80ms）对应接触时物理上的突然；松开（180ms）对应真实按钮机械回弹时的缓慢。辉光用不透明度交叉淡入淡出 `::before` 上静止的阴影和 `::after` 上按下的阴影，时间和大小变化相同，而不是直接给 `box-shadow` 做动画——那样每一帧都会重绘。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 按压深度 | 中等 | 按住时按钮变得有多小：轻是 97%，中等 95%，深是 90%；小于 90% 看起来像坏了 |
| 松开速度 | 普通 | 弹回要花多长时间：慢是 300ms，普通 180ms，快 110ms；大约是按压时间的两倍，感觉最自然 |
| 松开手感 | 回弹 | 回弹会稍微超过完整尺寸，再落定；平滑会减速到位；均匀保持一个恒定速度 |
| 按压速度 | 普通 | 缩小要花多长时间：慢是 130ms，普通 80ms，快 50ms；按压应该感觉立刻生效 |

## 生产注意事项
- **`pointercancel` 处理**：除了 `pointerup`，一定要监听 `pointercancel`。如果用户按下后开始滚动手势，`pointerup` 可能不会触发，按钮就会卡在按下状态。片段里还讲了另外两种卡住的办法：没有 `setPointerCapture`，指针移出按钮后就收不到松开；没有 `blur` 监听，按住键时用 Tab 把焦点移开，`keyup` 会发到下一个元素上。
- **按钮元素上的 `touch-action: manipulation`**：可以去掉移动浏览器上 300ms 的点按延迟，不需要单独的快速点按库。
- **阴影配对**：把阴影缩小和缩放变化配在一起，真实感更强——悬停时微微抬起、按下时落下的按钮，和实体按钮的行为一致。
- **`prefers-reduced-motion`**：为要求减弱动态效果的用户关掉过渡，这样按钮会立刻改变大小，而不是做动画；演示就是这样做的。
- **GSAP**：按下时用 `gsap.to(btn, { scale: 0.95, duration: 0.08, ease: "power1.in" })`；松开时用 `gsap.to(btn, { scale: 1, duration: 0.18, ease: "back.out(1.7)" })`。
- **Framer Motion**：`<motion.button whileTap={{ scale: 0.95 }} />`——一个属性就够了，按压和松开自动处理。

## 另见
- [点击涟漪](../click-ripple/) — 从按下的位置扩散开的涟漪
- [悬停状态动画](../hover-state/) — 点击之前项目就有了反应
- [对勾绘制](../checkmark-draw/) — 任务完成后自己画出来的对勾
