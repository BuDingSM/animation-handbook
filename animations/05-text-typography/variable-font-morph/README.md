# 可变字体变形

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/variable-font-morph/)

## 它是什么
可变字体把一整套字重（从细到特粗）装进一个文件里，每种样式只是一个数字。正因为这样，浏览器可以平滑地从一种样式移到另一种样式，所以一个词可以在你眼前变粗或变细。在演示里，这个词在网站字体的常规字重和特粗字重之间移动，还会通过倾斜来模拟斜体，因为这个字体本身没有斜体样式。

## 什么时候用
- 循环呼吸或跳动的主视觉标题
- 悬停效果，字重增加表示可以互动
- 加载状态，用字重循环代替转圈
- 品牌表达，可变轴本身就是视觉识别的一部分
- 互动滑块，让用户实时配置字重/字宽/样式

## 工作原理
在元素上设置 `font-variation-settings` 并加上 CSS 过渡。浏览器会同时插值所有的轴：

```css
.headline {
  font-weight: 400;
  font-variation-settings: 'wght' 400;
  transition: font-weight 600ms cubic-bezier(.4, 0, .2, 1),
              font-variation-settings 600ms cubic-bezier(.4, 0, .2, 1);
}

.headline:hover {
  font-weight: 800;
  font-variation-settings: 'wght' 800;
}
```

对于 JavaScript 驱动的变形，更新一个 CSS 自定义属性：

```js
function apply(wght, slnt) {
  const el = document.documentElement;
  el.style.setProperty('--wght', wght);   // 400–900 for the site font
  el.style.setProperty('--slnt', slnt);   // 0 to –15, drawn as a tilt
}
```

```css
.headline {
  font-weight: var(--wght);
  font-variation-settings: 'wght' var(--wght);
  transform: skewX(calc(var(--slnt) * 1deg));   /* the site font has no slant axis */
}
```

对于在预设之间循环的变形，用 `setInterval`：

```js
const PRESETS = [
  { wght: 400, slnt:   0 },
  { wght: 900, slnt:   0 },
  { wght: 700, slnt: -15 },
];
let i = 0;
setInterval(() => { apply(...Object.values(PRESETS[i++ % PRESETS.length])); }, 1400);   // Speed + an 800ms rest
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 每次变化需要的时间：慢速 1000ms，普通 600ms，快速 350ms；然后这个词休息 800ms 再进行下一次变化 |
| 变化时倾斜 | 开 | 有些变化还会让词最多倾斜 15 度；关掉的话，只变字重 |
| 你的文字 | Type | 会变的词 |

## 生产注意事项
- **轴的范围因字体而异**：`wght` 100–900 是标准，但 Recursive 可以到 1000，有些字体从 200 开始。一定要查字体的文档，或者用可变字体检查工具（例如 Wakamai Fondue，wakamaifondue.com）。
- **`font-variation-settings` 是全有或全无**：如果你设置了 `font-variation-settings: 'wght' 700`，其他所有的轴都会重置为默认值。一定要把你在意的轴全部写出来，哪怕没有变化。
- **字重优先用 CSS `font-weight`**：`font-weight: 700` 对可变字体有效，而且比 `font-variation-settings: 'wght' 700` 更易读。只有在非标准轴（`CASL`、`MONO` 等）上才用 `font-variation-settings`。
- **过渡性能**：现代浏览器里可变字体的插值由 GPU 文字渲染管线处理。它不如 `transform`/`opacity` 动画快，但对单个展示元素来说，在中端设备上一般都很流畅。
- **字体加载**：演示用的是网站自己的字体 Schibsted Grotesk，每个页面都已经加载了它，而且它的字重从 400 到 900 是可变的，所以字重可以平滑地离线变形。它没有斜体轴，所以倾斜是用 `skewX()` 实现的。轴更多的字体能给你更多可变的东西：比如 Recursive 就增加了 Casual 和斜体轴。用 `@font-face` 和 `font-display: swap` 自己托管它。
- **推荐的可变字体**：Recursive（CASL、MONO、slnt、wght）、Inter（wght）、Fraunces（opsz、SOFT、WONK、wght）、Bricolage Grotesque（opsz、wdth、wght）。
- **库**：GSAP 像补间其他值一样补间 `fontVariationSettings` 和 `fontWeight`，这在变形需要和其他动画保持同步时有帮助；单独使用时，一个 CSS 过渡就够了，就像这里这样。

## 另见
- [文字渐变动画](../text-gradient-animation/) — 颜色在静止的文字中流动
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [描边填充](../outline-to-fill/) — 空心字以擦除或淡入的方式填满颜色
