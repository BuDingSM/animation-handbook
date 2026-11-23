# 手风琴展开/收起

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/accordion/)

## 它是什么
手风琴是一叠标题，每个标题都能打开，在下方显示更多内容，然后再收起来。难的是这个动作：内容的盒子必须从零长到自然高度，而浏览器本身做不到这一点。演示展示了两种做法：先测好内容的高度，或者让网格行从零长到完整尺寸。

## 什么时候用
- 常见问题区块和文档页面
- 带分组选项的设置面板
- 带嵌套子项的导航菜单
- 任何不需要一次全部显示的分层内容

## 工作原理
**做法 1——JavaScript 实测高度：**

在过渡开始前先测出 `scrollHeight`（包括溢出部分的完整内容高度），把它明确地设置好，然后再动画到这个高度：

```js
function openItem(item) {
  const body  = item.querySelector('.acc-body');
  const inner = item.querySelector('.acc-inner');

  const targetHeight = inner.scrollHeight;
  body.style.setProperty('--target-h', targetHeight + 'px');
  body.classList.add('open');
}
```

```css
.acc-body {
  overflow: hidden;
  height: 0;
  transition: height 300ms ease-in-out;
}
.acc-body.open {
  height: var(--target-h);
}
```

演示会保留这个像素高度，并在窗口大小变化时，以及在网站字体加载完成后重新测量，这样打开的答案永远不会被裁剪。像下面生产注意事项里说的那样，在过渡结束后把 `height` 设为 `auto`，内容就能自己重新排布了。

**做法 2——CSS `grid-template-rows`：**

不需要 JavaScript。把内容包进一个网格容器，把行从 `0fr` 动画到 `1fr`：

```css
.acc-body {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 300ms ease-in-out;
}
.acc-body.open {
  grid-template-rows: 1fr;
}

.acc-inner {
  overflow: hidden;
  min-height: 0; /* required for 0fr to work */
}
```

用 JS 切换 `.open` 类即可——不需要测量：

```js
trigger.addEventListener('click', () => {
  item.classList.toggle('open');
  body.classList.toggle('open');
  trigger.setAttribute('aria-expanded', item.classList.contains('open'));
});
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 答案打开或收起要花多长时间：慢是 500ms，普通 300ms，快 180ms |
| 手感 | 轻柔 | 轻柔是缓入缓出，打开收起都合适；平滑在结尾放慢；均匀保持一个恒定速度 |
| 可同时打开多个 | 关 | 关：打开另一个时会关掉当前打开的答案，列表保持简短，但如果关掉的答案在上面，被按的问题会向上移动；开：每个答案一直打开，直到再次按下它的问题，这样什么都不会向上跳：被按的问题留在原地，下面的问题只向下滑动让出位置 |
| 实现方式 | 实测高度 | 实测高度先测量答案，再长到这个高度；网格行让网格行从零长到完整尺寸，无需测量；两者看起来一样 |
| 箭头翻转 | 开 | 答案打开时，每个问题旁边的箭头会转向上方 |
| 文字淡入 | 关 | 答案的文字在盒子打开时淡入并微微上升，每段比上一段晚一会儿 |

## 生产注意事项
- **绝不要把 `max-height` 动画到一个很大的值**：缓动会先跑过用不上的空间，让时间变得不可预测，动画感觉头重脚轻。始终动画真实高度。
- **CSS 网格做法的浏览器支持**：`grid-template-rows` 动画在 Chrome 107+、Firefox 107+、Safari 16+ 中可用。旧浏览器请回退到 JS 测量做法。
- **打开后把 `height` 设为 `auto`**：对于 JS 做法，监听 `transitionend`，然后把 `height` 设为 `auto`，这样内容可以重新排布（比如用户调整窗口大小时）。收起前先恢复成明确的像素值。
- **无障碍要求**：触发器上加 `aria-expanded`，内容面板上加 `id`，用 `aria-controls` 把两者连起来，内容上加 `role="region"`，方便地标导航。
- **展开时错开出现**：身体里的子元素可以在高度动画进行中，带着错开的延迟淡入——见演示里的「文字淡入」设置。
- **Radix UI Accordion**：完全无障碍、键盘可操作，可以通过 `data-state="open"/"closed"` 属性做动画。Framer Motion 的 `AnimatePresence` 能处理条件渲染内容的进出场。
- **为什么演示要给高度做动画**：答案的盒子长大本身就是这个效果，所以演示直接给它的高度（或网格行）做动画，这是只动画 transform 和 opacity 的例外；一次只打开一个短面板，并把视频、大图片等重内容放在面板外面、远离会动的东西，让它保持轻量。

## 另见
- [开关滑动](../toggle-switch/) — 简单的开和关，没有高度变化
- [抽屉滑出](../drawer-slide/) — 在页面上滑入的面板
- [FLIP 技巧](../../03-page-transitions/flip-technique/) — 布局变化时让其他元素平滑移动
