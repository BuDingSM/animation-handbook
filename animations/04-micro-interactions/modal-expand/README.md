# 弹窗展开

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/modal-expand/)

## 它是什么
弹窗展开让对话框从打开它的按钮处长出来。页面量出那个按钮的位置，让窗口从那个精确的点长到屏幕中间的全尺寸，这样窗口来自哪里、关闭时回到哪里，都很清楚。

## 什么时候用
- 点按卡片打开详情弹窗的卡片操作
- 展开成表单或面板的浮动操作按钮（FAB）
- 任何按钮和弹窗之间的空间关系对理解很重要的触发器
- 模仿 iOS 长按 → 上下文菜单展开的应用式界面

## 工作原理
点击时，用 `getBoundingClientRect()` 量出按钮的位置，再算出相对于弹窗自己角落的同一个点：

```js
function openModal(btn) {
  const sr = stage.getBoundingClientRect();
  const br = btn.getBoundingClientRect();
  // The button's center, relative to the stage
  const bx = br.left + br.width / 2 - sr.left;
  const by = br.top + br.height / 2 - sr.top;
  // The same point, relative to the modal's laid-out corner (offsetLeft/Top ignore its scale)
  modal.style.transformOrigin = `${bx - modal.offsetLeft}px ${by - modal.offsetTop}px`;
  modal.classList.add('open');
}
```

```css
:root {
  --modal-dur: 320ms;
  --modal-ease: cubic-bezier(.34, 1.3, .64, 1);
  --start-scale: 0.1;
}

.modal {
  position: absolute;
  inset: 0; margin: auto;              /* centered without a transform */
  width: clamp(240px, 55%, 320px); height: fit-content;
  transform: scale(var(--start-scale));
  opacity: 0;
  transition: transform var(--modal-dur) var(--modal-ease), opacity 200ms ease;
  pointer-events: none;
}
.modal.open { transform: scale(1); opacity: 1; pointer-events: auto; }
```

回弹缓动（`cubic-bezier(.34, 1.3, .64, 1)`）会造成一点过头，让弹窗在落定位置时有令人满意的“弹出”感。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 窗口长出来要花多长时间：慢是 500ms，普通 320ms，快是 200ms |
| 起始大小 | 小 | 窗口离开按钮时有多大：极小是全尺寸的 5%，小是 10%，中等 30%；太大，生长就很难看出来 |
| 手感 | 回弹 | 回弹稍微超过全尺寸，再落定；平滑慢慢停下；均匀保持一个稳定的速度 |
| 标记生长起点 | 开 | 在窗口长出来的点上显示一个小圆点 |

## 生产注意事项
- **调整大小时重新计算 `transform-origin`**：如果用户在两次打开之间调整窗口大小，缓存的原点就过时了。在点击处理函数里重新计算，不要在挂载时算。
- **关闭动画**：关闭时反向做这个效果——向原点缩回去。这需要记住上次用哪个按钮打开了弹窗。
- **焦点管理**：打开时把焦点移进弹窗（`modal.querySelector('button, input, [tabindex]')?.focus()`）。关闭时把焦点还给触发按钮。
- **`<dialog>` 元素**：原生的 `<dialog>` 元素自动处理焦点捕获和 Esc 键。用 `::backdrop` 做遮罩动画，用 `@starting-style`（Chrome 117+）做进入动画，不用 JavaScript 切换类。
- **Framer Motion**：`<motion.div initial={{ scale: 0.1, originX: "50%", originY: "50%" }} animate={{ scale: 1 }} />`。动态原点时，把计算出的值作为 motion 属性传进去。
- **GSAP**：`gsap.fromTo(modal, { scale: 0.1, transformOrigin: `${ox}px ${oy}px` }, { scale: 1, duration: 0.32, ease: "back.out(1.5)" })`。

## 另见
- [抽屉滑出](../drawer-slide/) — 从边缘滑入的面板
- [提示气泡浮现](../tooltip-reveal/) — 简短说明的小标签
- [FLIP 技巧](../../03-page-transitions/flip-technique/) — 同样的先量后移思路，用于任何布局变化
