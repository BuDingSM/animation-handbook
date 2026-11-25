# 搜索框展开

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/expanding-search/)

## 它是什么
展开搜索在需要之前只显示一个搜索图标。按下图标，搜索框从图标处打开，盖住旁边的链接，并把光标放进去；Esc、关闭按钮或点击别处会再关掉它。它在拥挤的工具栏里省地方，同时让搜索一按就到。

## 什么时候用
- 地方紧张的工具栏和页眉
- 移动端页眉，完整的搜索框会挤占标志和菜单
- 时不时用搜索的文档和帮助网站
- 一条栏里有很多控制的仪表盘和管理工具

## 工作原理
输入框从一开始就按完整大小布局，大小永远不变。一个带圆角的 `clip-path` 藏起它全部，只留图标上方一个 44px 的圆；打开时动画裁剪，直到整个输入框显示，从图标那侧开始，所以旁边的东西都不用重新布局。`visibility` 跟着裁剪走：打开时立刻开启，关闭时等关闭裁剪结束才关闭，所以关掉的输入框既不能点击，也不能用 Tab 到达：

```css
.sfield {
  position: absolute; inset: 0;                          /* full size, over the links */
  visibility: hidden;
  clip-path: inset(0 0 0 calc(100% - 44px) round 22px); /* just the circle over the icon at the right end */
  transition: clip-path 280ms cubic-bezier(.22, 1, .36, 1), visibility 0s linear 280ms;
}
.to-right .sfield { clip-path: inset(0 calc(100% - 44px) 0 0 round 22px); } /* the icon at the left end */
.bar.open .sfield {
  visibility: visible;
  clip-path: inset(0 round 22px);                        /* all of it */
  transition: clip-path 280ms cubic-bezier(.22, 1, .36, 1), visibility 0s;
}
```

```js
function open() { bar.classList.add('open'); icon.setAttribute('aria-expanded', 'true'); icon.inert = true; input.focus(); }
function close(focusIcon) {
  bar.classList.remove('open'); icon.setAttribute('aria-expanded', 'false'); icon.inert = false;
  if (focusIcon) icon.focus();                          // after Escape or the close button
}
icon.addEventListener('click', open);
closeButton.addEventListener('click', () => close(true));
field.addEventListener('keydown', e => { if (e.key === 'Escape') close(true); });
field.addEventListener('focusout', e => {             // leaving it: the focus went somewhere outside the field
  if (closesOnLeave && !field.contains(e.relatedTarget)) close(false);
});
```

图标留在输入框上方。输入框打开时，图标的背景变透明，它的放大镜旋转淡出，关闭按钮的叉号在它下面转进来，所以一个看起来变成了另一个；输入框下面的链接淡出。输入框打开时，图标让按压穿过，并且是 `inert` 的，Tab 就落不到藏起来的东西上；输入框里面有焦点时，用内嵌的圆环显示它的焦点。在输入框周围（它的放大镜或边缘）按一下会取消 `mousedown`，把光标放回框里，这样就不算离开。「演示一下」会打开输入框并输入，但不移动真正的焦点，所以永远不会打开手机的键盘；运行时按压或按 Tab 进入输入框，会把输入框交给访客，仍然开着。减弱动态效果下，过渡会被关掉，输入框立刻打开和关闭。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 展开方向 | 左 | 左：把图标放在工具栏右端，输入框向左长出来，盖住链接；右：把图标放在标志旁边，输入框向右长出来 |
| 速度 | 普通 | 输入框打开或关闭要花多长时间：慢是 450ms，普通 280ms，快 160ms；再慢，打字就得等它 |
| 离开时收起 | 开 | 开：焦点一离开就关掉输入框，无论是点别处还是按 Tab；关：一直开着，直到按 Esc 或关闭按钮 |

## 生产注意事项
- **绝不要动画宽度**：变大的 `width` 让浏览器每一帧都重新布局整个工具栏，旁边的项目会抖动，慢手机会掉帧。裁剪（或 `scaleX()` 背景层，文字在它后面淡入）只重绘输入框。
- **手机**：iOS Safari 会在文字小于 16px 的输入框上把页面放大，所以演示在触屏上用 16px。很窄的屏幕上，可以考虑把搜索打开成盖住整个页眉的横条。
- **保留键盘的位置**：Esc 或关闭按钮后把焦点还给图标，关掉的输入框不要留在 Tab 顺序里（关闭裁剪后的 `visibility: hidden`，或 `inert`），输入框打开时让被盖住的图标变成 `inert`。
- **读屏软件的名字**：图标按钮需要名字（“搜索”）和 `aria-expanded`，输入框需要标签（视觉隐藏也可以），容器需要 `role="search"` 或 `<form role="search">`。
- **留下的文字**：决定离开一个有文字的输入框会怎样。演示会关掉它，下次保留文字；有些网站会让有文字的输入框保持打开。
- **GSAP**：`gsap.fromTo(field, { clipPath: "inset(0% 0% 0% 90% round 22px)" }, { clipPath: "inset(0% 0% 0% 0% round 22px)", duration: 0.28, ease: "power3.out" })`；两端保持同样的单位，这样值才能混合。
- **Framer Motion**：`<motion.div animate={{ clipPath: open ? "inset(0% 0% 0% 0% round 22px)" : "inset(0% 0% 0% 90% round 22px)" }} transition={{ duration: 0.28 }} />`。

## 另见
- [汉堡菜单切换](../hamburger-menu-toggle/) — 菜单打开时图标变成叉号
- [表单字段变形](../form-field-morph/) — 输入时字段的标签让开
- [焦点环动画](../focus-ring/) — 显示键盘位置的圆环
- [抽屉滑出](../drawer-slide/) — 需要时滑入的隐藏面板
