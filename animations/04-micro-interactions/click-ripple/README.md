# 点击涟漪

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/click-ripple/)

## 它是什么
涟漪是一个柔和的圆圈，从你按下的位置长出来，再淡出。它确认按压已经登记，并精确显示落在哪里，在触屏上最有帮助——触屏上悬停什么都不会触发。Material Design，Google 的设计系统，让它流行起来。

## 什么时候用
- 主要操作按钮（提交、确认、购买）
- 响应点按的列表项和菜单项
- 任何用户需要立刻确认触摸已登记的表面
- 涟漪能避免“到底按上了吗？”式停顿的图标按钮

## 工作原理
在 `pointerdown` 时，用 `getBoundingClientRect()` 测量点击相对于按钮的位置，生成一个绝对定位的 `<span>`，并把它从 `scale(0)` 动画到很大的缩放，同时把不透明度淡到 0：

```js
function spawnRipple(btn, e) {
  const r = btn.getBoundingClientRect();
  const size = Math.max(r.width, r.height) * 2;
  const x = e.clientX - r.left;
  const y = e.clientY - r.top;

  const el = document.createElement('span');
  el.style.cssText = `
    position: absolute;
    width: ${size}px; height: ${size}px;
    left: ${x - size/2}px; top: ${y - size/2}px;
    border-radius: 50%;
    background: rgba(255,255,255,0.4);
    transform: scale(0);
    animation: ripple 600ms ease-out forwards;
    pointer-events: none;
  `;
  btn.appendChild(el);
  el.addEventListener('animationend', () => el.remove(), { once: true });
}
```

```css
@keyframes ripple {
  to { transform: scale(2.5); opacity: 0; }
}
```

按钮需要 `position: relative; overflow: hidden` 来装住涟漪。

键盘按压没有位置：Enter 或空格键会触发一个 `detail` 为 0 的 `click`，这个点击会从按钮中间开始涟漪。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 起始位置 | 按下的位置 | 按下的位置精确显示按压落在哪里；中间更简单，但没那么精确 |
| 速度 | 普通 | 涟漪扩散淡出要花多长时间：慢是 1000ms，普通 600ms，快 350ms；比悬停变化稍长一点，因为点击已经完成了 |
| 涟漪大小 | 中等 | 圆圈在淡出前长到多大：小、中、大最后分别是按钮较长边的 3、5、8 倍宽；即使是小，也能到达按钮的每个角落 |
| 涟漪强度 | 中等 | 涟漪刚开始时有多明显：淡淡是 20%，中等 40%，强烈是 60% |
| 涟漪颜色 | 白色 | 白色适合彩色按钮，橙色配主题色，黑色适合浅色按钮 |

## 生产注意事项
- 按钮上的 **`overflow: hidden`** 是必需的——没有它，涟漪会超出按钮边界。
- 涟漪元素上的 **`pointer-events: none`** 能防止它干扰随后快速触发的点击。
- **触摸事件**：`pointerdown` 对鼠标和触摸都有效。不要用 `mousedown`——它在触摸时不会触发。
- **清理**：一定要在 `animationend` 里移除元素。压力测试（快速点击）中，没有清理，DOM 节点会迅速积累。
- **减弱动态效果**：跳过长大的圆圈，改成在按钮上淡入一片平坦的高亮，这样按压仍然有显示。演示就是这样做的。
- **GSAP**：`gsap.fromTo(el, { scale: 0 }, { scale: 2.5, opacity: 0, duration: 0.6, ease: "power2.out", onComplete: () => el.remove() })`。
- **Material Web Components**：`<md-ripple>` 组件会自动处理所有这些，包括触摸和键盘激活。

## 另见
- [按钮按压缩放](../button-press-scale/) — 按下时按钮会缩小
- [悬停状态动画](../hover-state/) — 点击之前项目就有了反应
- [对勾绘制](../checkmark-draw/) — 任务完成后自己画出来的对勾
