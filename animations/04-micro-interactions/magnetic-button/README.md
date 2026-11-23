# 磁吸按钮

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/magnetic-button/)

## 它是什么
磁吸按钮好像被指针吸住。指针靠近时，按钮向它移动一部分，标签再多移一点，所以标签像浮在按钮上面一样。指针离开时，按钮弹回来；它在布局里的位置永远不会变。

## 什么时候用
- 作品集和落地页上的主视觉 CTA，一个按钮承载大部分注意力
- 想让它感觉灵敏、“活着”又不吵闹的主要操作
- 触感指针玩法是品牌一部分的获奖风格 / 代理网站
- 要克制——一屏一两个磁吸元素，不要每个按钮都这样

## 工作原理
每次 `pointermove` 经过舞台，你都量出从按钮中心到指针的向量。如果距离在激活半径内，就按一个强度系数缩放这个向量来移动按钮，并做衰减，让吸附在半径边缘处缓到零。按钮里面的标签再移动同样偏移的 35%，所以它比按钮走得远一点，像浮在按钮上面一样。吸附是一个接受舞台上一点的函数，所以鼠标和手指都能调用它：

```js
function pull(px, py) {
  const s = stage.getBoundingClientRect(), b = btn.getBoundingClientRect();
  const bx = b.left + b.width/2 - s.left, by = b.top + b.height/2 - s.top;
  const dx = px - bx, dy = py - by;
  const dist = Math.hypot(dx, dy);
  if (dist < radius) {
    const f = 1 - dist / radius;                 // 1 at center → 0 at edge
    const tx = dx * strength * f, ty = dy * strength * f;
    btn.style.setProperty('--tx', tx + 'px');    // .mag { transform: translate(var(--tx), var(--ty)) }
    btn.style.setProperty('--ty', ty + 'px');
    label.style.transform = `translate(${tx*0.35}px, ${ty*0.35}px)`;
  } else { home(); }
}
function home() {
  btn.style.removeProperty('--tx'); btn.style.removeProperty('--ty');
  label.style.transform = '';
}
const pointAt = e => {
  const s = stage.getBoundingClientRect();
  pull(e.clientX - s.left, e.clientY - s.top);
};

// Mouse: pull while it moves over the stage, spring home when it leaves
stage.addEventListener('pointermove', e => { if (e.pointerType === 'mouse') pointAt(e); });
stage.addEventListener('pointerleave', home);

// Finger or pen: pull while it touches the stage, spring home when it lifts or leaves the stage
let touchId = null;
stage.addEventListener('pointerdown', e => {
  if (e.pointerType === 'mouse') return;
  touchId = e.pointerId; pointAt(e);
});
stage.addEventListener('pointermove', e => { if (e.pointerId === touchId) pointAt(e); });
['pointerup', 'pointercancel', 'pointerleave'].forEach(type => stage.addEventListener(type, e => {
  if (e.pointerId === touchId) { touchId = null; home(); }
}));
```

弹回只是按钮上的一段 CSS `transition: transform .55s cubic-bezier(.22,1,.36,1)`——JS 清掉偏移后，按钮自己缓回原位。触屏上不能悬停，同样的吸附会在手指按住舞台时跟随手指，手指抬起时按钮弹回原位。

键盘驱动的是同一个 `pull()`：按钮聚焦时，一个假想的指针从它的中心开始，每按一次方向键移动作用范围的十分之一，按钮向它倾斜，超出范围后弹回原位。Esc，或把焦点移开，会把它送回家；空格键或 Enter 像指针一样按下它。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 吸附力度 | 中等 | 按钮向指针移动多远：轻柔是距离的 20%，中等 35%，强烈 50%（指针靠近时），越远占比越小，到作用范围的边缘为零；超过大约一半会感觉松散 |
| 作用范围 | 中等 | 指针靠多近，吸附才开始：短是 100px，中等 160px，长是 240px；太远，按钮会对无关的移动做出反应 |

## 生产注意事项
- **按悬停能力门控**：只在 `window.matchMedia('(hover: hover)').matches` 的地方绑定悬停磁吸。触屏上 `pointermove` 只在手指按下时触发，所以手指按下时向触摸点吸附，抬起时把按钮送回家，或退回到普通的按压缩放。
- **只用变换**：动画 `translate`，永远不要用 `top`/`left`。按钮留在合成器上，布局永远不回流，所以即使指针快速移动也能保持 60fps。
- **防抖没必要，但 rAF 有帮助**：场景更重时，把变换写入批量放进一个 `requestAnimationFrame` 里，这样一帧里的多次 `pointermove` 事件会合并成一次样式写入。
- **减弱动态效果**：把过渡缩短成快速、均匀的移动，这样按钮仍会跟随指针，没有缓慢的滑行。
- **GSAP**：`gsap.quickTo(btn, "x", {duration:.5, ease:"power3"})` 是地道的写法——它每一帧都向目标插值，而不是跳过去，免费获得更平滑的滞后。
- **Framer Motion**：用一个 `useSpring` 跟随指针偏移，传给 `motion.button` 的 `x`/`y`；弹簧配置取代 CSS 缓动。

## 另见
- [按钮按压缩放](../button-press-scale/) — 按下时它也会做的小收缩
- [悬停状态动画](../hover-state/) — 对指针做出反应的更简单方式
- [光标跟随](../cursor-follower/) — 一个跟随指针移动的形状
