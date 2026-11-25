# 悬停状态动画

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/hover-state/)

## 它是什么
悬停状态是当指针移到可点击的东西（比如按钮、链接或卡片）上时播放的小变化。它在人们点击之前就告诉他们，这个项目会有反应。演示展示了六种常见做法：变色、稍微变大、带阴影的抬起、画出的下划线、向前轻推的箭头和扫过的一层色彩。

## 什么时候用
- 需要表示可点击的导航链接和按钮
- 网格里的卡片，悬停可以预览操作
- 图标按钮，悬停会显示标签或改变图标颜色
- 任何只靠默认光标不足以传达交互的元素

## 工作原理
六种技巧都用 CSS 的 `transition`，由 `:hover` 驱动（或用 `:active` 作为触摸的替代）。一个 `--dur` 和一个 `--ease` CSS 变量同时控制每张卡片：

```css
:root { --dur: 180ms; --ease: ease; }

/* Color shift */
.h-color { transition: background var(--dur) var(--ease), border-color var(--dur) var(--ease); }
@media (hover: hover) { .h-color:hover { background: #2a201a; border-color: var(--ui-accent); } }
.h-color:active, .h-color:focus-visible { background: #2a201a; border-color: var(--ui-accent); }

/* Scale */
.h-scale { transition: transform var(--dur) var(--ease); }
@media (hover: hover) { .h-scale:hover { transform: scale(1.03); } }
.h-scale:active, .h-scale:focus-visible { transform: scale(1.03); }

/* Background sweep */
.h-sweep::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(255,157,92,.13);
  transform: translateX(-100%);
  transition: transform var(--dur) var(--ease);
}
@media (hover: hover) { .h-sweep:hover::before { transform: translateX(0); } }
```

`@media (hover: hover)` 门控防止悬停样式在触屏设备上粘住，而 `:active` 提供点按的替代。

每张卡片也可以获得键盘焦点（`tabindex="0"`），`:focus-visible` 和 `:active` 放在一起，在悬停门控之外，所以按 Tab 会一张一张显示同样的变化；焦点离开时，卡片回到静止状态。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 每次变化要花多长时间：慢是 300ms，普通 180ms，快是 110ms；悬停变化保持在 200ms 或更短，这样滑过多个项目时永远不会感觉黏滞 |
| 手感 | 平滑 | 平滑减速到位；回弹稍微过头，再落定；轻柔缓入缓出；均匀保持一个稳定的节奏 |

## 生产注意事项
- **时长上限**：超过 200ms 的悬停动画，会让光标快速滑过多个元素时感觉迟钝、发“黏”。保持在 180ms 或更短。
- **触摸替代**：`:hover` 在触摸上不可靠。用 `@media (hover: hover)` 门控只在悬停时用的样式，并加上 `:active` 的等价样式作为点按反馈。
- **背景扫过的做法**：用 `::before`，父级设 `overflow: hidden`，用 `translateX` 而不是 `width`——`width` 会触发布局，`transform` 不会。
- **GSAP**：在 `mouseenter` / `mouseleave` 时 `gsap.to(el, { scale: 1.03, duration: 0.18 })`。对简单的悬停来说太重了；CSS 原生就能处理。
- **Framer Motion**：`<motion.div whileHover={{ scale: 1.03 }} />`——地道的 React 等价写法。

## 另见
- [按钮按压缩放](../button-press-scale/) — 按下时按钮缩小
- [提示气泡浮现](../tooltip-reveal/) — 指向一个项目会显示一条简短说明
- [点击涟漪](../click-ripple/) — 涟漪从你按下的位置扩散
- [聚光灯悬停辉光](../spotlight-hover/) — 柔和的光跟随指针，穿过一排卡片
