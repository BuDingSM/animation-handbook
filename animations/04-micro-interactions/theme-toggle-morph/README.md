# 主题切换变形

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/theme-toggle-morph/)

## 它是什么
主题切换变形是一个图标，在网站从浅色切换到深色时，从太阳变成月牙。太阳的光线缩进中心，同时一个隐藏的圆圈滑过太阳，把它切成月牙，所以两种状态明显是同一个形状在变化。演示里，一张小的预览卡片同时翻到它的深色面。

## 什么时候用
- 在浅色和深色模式之间切换界面的页眉或设置控件。
- 任何二元外观切换，两个选项有自然的图标（太阳/月亮、开/关、白天/夜晚）。
- 普通复选框会感觉突兀、短暂变形让状态变化清晰、又没有整页闪烁的地方。

控件放在密集表单里，或用户可能在没有旁边文字标签时认不出太阳/月亮是“主题”时，改用普通的带标签开关。

## 工作原理
图标是一个内联 SVG：一个填满的 `disc`、一组八条 `line` 光线，和一个装着可移动 `cutout` 圆圈的 `mask`。浅色状态下，镂空圆圈停在圆盘外面，所以圆盘完整渲染，光线可见。切换会加上一个 `.dark` 类，把光线从图标中心缩放到零，并把镂空圆圈滑过圆盘——遮罩减去重叠的部分，留下一个月牙。一张单独的翻转卡片用同样的时长和缓动旋转。光线、圆盘、镂空圆圈和卡片只改变 `transform` 和 `opacity`。图标的颜色通过按钮上的 `color` 过渡，从橙色淡到淡紫色，SVG 用 `currentColor` 接收它。

```css
.icon line{transform-origin:12px 12px;
  transition:transform var(--morph-dur) var(--morph-ease),opacity var(--morph-dur) var(--morph-ease)}
.icon .cutout{transform:translate(20px,-20px);      /* parked off the disc */
  transition:transform var(--morph-dur) var(--morph-ease)}
.demo.dark .icon line{transform:scale(0);opacity:0} /* rays retract */
.demo.dark .icon .cutout{transform:translate(7px,-4px)} /* slides in → crescent */
```

```html
<circle class="disc" cx="12" cy="12" r="5" mask="url(#moon-mask)"/>
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 变化要花多长时间：慢是 800ms，普通 500ms，快是 300ms；不到大约 200ms，月牙形成得太快看不清；超过 800ms 会拖沓 |
| 手感 | 轻柔 | 轻柔缓入缓出；回弹稍微过头，再落定；均匀保持一个稳定的速度 |

## 生产注意事项
控件是一个真正的 `<button>`，它的 `aria-label` 说出*操作*（“切换到深色主题”），每次按下都会变化，这样读屏软件会说出按下它会做什么；不要用 `aria-pressed`，因为切换的名称不能变，“切换到浅色主题，已按下”自相矛盾（用 `aria-pressed` 时，用一个固定的名字，比如“深色主题”）。预览卡片是 `aria-hidden`，因为它只是说明效果。第一次加载时，读取用户保存的偏好，退回到 `prefers-color-scheme`：`const dark = localStorage.getItem('theme') === 'dark' || (localStorage.getItem('theme') === null && matchMedia('(prefers-color-scheme: dark)').matches)`，然后在按钮可交互之前设置类和标签，避免闪烁。点击时把选择存到 `localStorage`，这样重新加载后还在。遵循 `prefers-reduced-motion: reduce`，去掉过渡——图标仍会跳到正确的状态，只是不做变形。Feather 之类的图标集把太阳和月亮作为分开的 SVG 提供；这里基于遮罩的变形，正是 Framer Motion 或 GSAP 会在同一条时间线上补间遮罩偏移和光线缩放来驱动的东西。

## 另见
- [开关滑动](../toggle-switch/) — 在开和关之间滑动的开关
- [汉堡菜单切换](../hamburger-menu-toggle/) — 三条线变成叉号
- [对勾绘制](../checkmark-draw/) — 自己画出来的对勾
