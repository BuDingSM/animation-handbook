# 提示气泡浮现

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/tooltip-reveal/)

## 它是什么
提示气泡是一个小标签，在你指向一个项目或用键盘到达它时出现在它旁边，给出页面上放不下的简短说明。它淡入并非常轻微地长大，显示前会等一会儿，这样指针只是经过东西时，提示不会闪现。指针离开后，它会停留一会儿再淡出，不会在指针刚滑出项目时立刻消失。

## 什么时候用
- 没有可见文字标签的图标按钮
- 需要在悬停时显示完整内容的截断文字
- 带验证规则或格式要求的表单字段
- 悬停时显示精确值的数据可视化元素（图表条、图形节点）

## 工作原理
提示气泡相对于触发器绝对定位，一开始看不见。悬停时，它在 `setTimeout` 延迟后淡入并缩放进来。鼠标离开后，它在短暂的第二次延迟后隐藏，这样指针刚滑出触发器时它不会立刻消失：

```css
.tooltip {
  position: absolute;
  bottom: calc(100% + 8px);
  left: 50%;
  transform: translateX(-50%) scale(0.95);
  transform-origin: bottom center;
  background: #1c1c22;
  border: 1px solid rgba(255, 255, 255, 0.09);
  color: #f4f4f2;
  padding: 7px 10px;
  border-radius: 6px;
  font-size: 11px;
  line-height: 1.5;
  width: max-content;
  max-width: 200px;
  z-index: 10;
  opacity: 0;
  pointer-events: none;
  transition: opacity 150ms ease, transform 150ms ease;
}

.tooltip.visible {
  opacity: 1;
  transform: translateX(-50%) scale(1);
}
```

```js
let showTimer, hideTimer;

trigger.addEventListener('pointerenter', () => {
  clearTimeout(hideTimer);
  showTimer = setTimeout(() => tip.classList.add('visible'), 300);
});

trigger.addEventListener('pointerleave', () => {
  clearTimeout(showTimer);
  hideTimer = setTimeout(() => tip.classList.remove('visible'), 100);
});

// Keyboard support
trigger.addEventListener('focusin', () => {
  clearTimeout(hideTimer);
  showTimer = setTimeout(() => tip.classList.add('visible'), 300);
});
trigger.addEventListener('focusout', () => {
  clearTimeout(showTimer);
  hideTimer = setTimeout(() => tip.classList.remove('visible'), 100);
});
```

Esc 会立刻隐藏显示的提示气泡，这样不用移动指针或焦点就能关闭它：

```js
document.addEventListener('keydown', e => {
  if (e.key !== 'Escape') return;
  clearTimeout(showTimer);
  tip.classList.remove('visible');
});
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 显示前延迟 | 中等 | 指针必须停留多长时间，提示气泡才会显示：无、短（150ms）、中等（300ms）或长（600ms）；300ms 让提示不会在指针经过时闪现，200ms 对很小的提示也可以 |
| 速度 | 普通 | 淡入和轻微长大（从它大小的 95%）要花多长时间：慢是 250ms，普通 150ms，快是 90ms |
| 隐藏前延迟 | 短 | 指针离开后，提示气泡停留多长时间：无、短（100ms）或长（300ms）；短暂的停顿让它不会在指针刚滑出项目时立刻消失 |
| 显示箭头 | 关 | 加一个小尖角，把提示气泡对准它的项目 |

## 生产注意事项
- **300ms 规则**：没有显示延迟，鼠标在页面上的每次移动都会触发提示闪现。300ms 是感觉灵敏又不烦人的最小值。提示很小时，200ms 也可以。
- **智能定位**：演示把提示气泡放在固定方向。生产实现需要感知视口的定位——提示会溢出边缘时翻转它。Floating UI（Atomics Design 出品）自动处理这个。
- **触屏设备**：触摸上提示没有悬停触发。长按或专门的信息按钮是触摸的等价物。演示用 `touchstart` 切换可见性作为替代。
- **提示上的 `pointer-events: none`** 让它不会接住指针，所以它永远不会挡住项目或它的邻居；代价是指针不能停在提示本身上面，WCAG 那条覆盖了这个。
- **WCAG 1.4.13（悬停或聚焦时的内容）**：提示必须不用移动光标就能关闭（比如 Esc 键），本身能被悬停而不消失，持续到光标移开。演示覆盖了“可关闭”：Esc 会立刻隐藏显示的提示气泡。隐藏延迟只有在提示接受指针时才对“可悬停”有帮助；演示的提示有 `pointer-events: none`，所以必须能悬停的提示需要打开指针事件。
- **Floating UI / Popper.js**：生产级定位库。`computePosition()` 加 `flip` 和 `shift` 中间件，处理所有边缘情况。
- **Radix UI Tooltip**：`<Tooltip.Root>`、`<Tooltip.Trigger>`、`<Tooltip.Content>`——完全无障碍、符合 WCAG 1.4.13、动画就绪。

## 另见
- [悬停状态动画](../hover-state/) — 指针在上面时有反应的项目
- [通知徽标脉冲](../badge-pulse/) — 吸引眼睛的脉冲圆点
- [弹窗展开](../modal-expand/) — 给提示气泡放不下的内容的完整窗口
