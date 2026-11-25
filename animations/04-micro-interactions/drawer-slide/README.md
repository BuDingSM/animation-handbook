# 抽屉滑出

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/drawer-slide/)

## 它是什么
抽屉是一块面板，在屏幕一边外面等着，按下按钮就滑入页面上方，通常装着菜单、筛选器或设置。它后面的页面通常会变暗，这样抽屉更显眼。到达时它会减速，像一块面板滑到停下；离开时加速，关起来比打开感觉更快。

## 什么时候用
- 汉堡按钮后面的移动端导航菜单
- 电商和搜索结果页上的筛选面板
- 设置、偏好和配置面板
- 随选中内容出现的上下文面板（VS Code 风格的侧边面板）

## 工作原理
抽屉一开始平移出屏幕外，过渡到自然位置。静止规则带着关闭的速度和曲线，打开状态带着打开的速度和曲线，所以每个方向都有自己的缓动：

```css
:root {
  --drawer-w: min(280px, 80%);
  --open-dur: 280ms;
  --close-dur: 220ms;
  --open-ease: ease-out;
  --close-ease: ease-in;
  --backdrop-op: 0.5;
}

.drawer {
  position: fixed;
  top: 0; left: 0; bottom: 0;
  width: var(--drawer-w);
  transform: translateX(-100%);
  transition: transform var(--close-dur) var(--close-ease);
  will-change: transform;
  z-index: 200;
}

.drawer.open {
  transform: translateX(0);
  transition: transform var(--open-dur) var(--open-ease);
}

/* Backdrop */
.backdrop {
  position: fixed; inset: 0;
  background: rgba(0,0,0,var(--backdrop-op));
  opacity: 0;
  transition: opacity var(--close-dur) var(--close-ease);
  pointer-events: none;
  z-index: 199;
}
.backdrop.visible {
  opacity: 1;
  pointer-events: auto;
  transition: opacity var(--open-dur) var(--open-ease);
}
```

用 Esc 键、点击背景和滑动手势关闭：

```js
function open() {
  drawer.classList.add('open');
  backdrop.classList.add('visible');
  drawer.inert = false;
}
function close() {
  drawer.classList.remove('open');
  backdrop.classList.remove('visible');
  drawer.inert = true; // a closed drawer cannot be reached with Tab
}

document.addEventListener('keydown', e => { if (e.key === 'Escape') close(); });
backdrop.addEventListener('click', close);

// Swipe-to-close
let dragStartX = 0;
drawer.addEventListener('pointerdown', e => { dragStartX = e.clientX; });
drawer.addEventListener('pointerup',   e => { if (e.clientX - dragStartX < -50) close(); });
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 滑入方向 | 左 | 抽屉从哪条边来：菜单和筛选器用左或右，底部动作表单用上或下 |
| 打开速度 | 普通 | 滑入要花多长时间：慢是 450ms，普通 280ms，快 170ms；到达时它会减速 |
| 关闭速度 | 普通 | 滑出要花多长时间：慢是 350ms，普通 220ms，快 130ms；离开时它会加速，稍短于打开感觉正好 |
| 页面变暗程度 | 中等 | 后面的页面变多暗：无、轻（25% 黑色）、中等（50%）或深（70%）；比大约 70% 更暗，感觉像对话框而不是抽屉 |

## 生产注意事项
- **焦点陷阱**：抽屉打开时，Tab 焦点必须在里面循环。用焦点陷阱库（比如 `focus-trap`），或原生 `<dialog>` 元素，它会自动困住焦点。
- **`aria-modal="true"` 和 `role="dialog"`**：读屏软件需要它们来播报抽屉是一个模态上下文。给抽屉标题加上 `aria-label` 或 `aria-labelledby`。
- **`will-change: transform`**：把抽屉提升到自己的合成层，防止滑动时重绘。动画完成后如果低端设备内存紧张，可以去掉 `will-change`。
- **右侧/底部抽屉**：筛选器从右侧滑入（`translateX(100%)`）；动作表单从底部滑入（`translateY(100%)`）。
- **触摸滑动关闭**：用 `pointerdown`/`pointermove`/`pointerup`（不要分开用鼠标/触摸事件）。测量位移，滑动距离超过关闭方向约 50px 就关掉。鼠标下给抽屉加 `user-select: none`，否则第一次拖动会选中它的文字，第二次就变成原生文字拖动。
- **Radix UI Sheet / shadcn Drawer**：完全无障碍、有动画的抽屉组件。Vaul（Emil Kowalski）为底部抽屉加了原生移动端风格的拖动关闭。

## 另见
- [弹窗展开](../modal-expand/) — 从按下的按钮处长出来的窗口
- [手风琴展开/收起](../accordion/) — 在原地打开的部分，而不是盖在页面上的
- [提示气泡浮现](../tooltip-reveal/) — 一条简短说明的小标签
