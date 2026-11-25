# 文字渐变动画

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/text-gradient-animation/)

## 它是什么
文字渐变动画用渐变而不是纯色来填充字母，然后在字母后面移动渐变，所以颜色看起来在词里流动，而字母保持静止。窍门是把渐变画成文字的背景，只在字母形状里显示这个背景，并让文字颜色本身透明。

## 什么时候用
- 创意、科技和品牌网站上的展示标题
- Logo 动画和字标
- 通过颜色循环表示活跃或加载状态的状态文字
- 需要视觉丰富度又不需要图片的装饰展示文字

## 工作原理
两个 CSS 属性让背景透过文字形状显示出来，然后一个关键帧动画移动背景：

```css
.gradient-text {
  /* The trick: two properties working together */
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;

  /* The gradient to animate */
  background-image: linear-gradient(90deg, #58a6ff, #56d364, #d2a8ff, #ffa657, #58a6ff);
  background-size: 300% 100%;
  animation: flow 4s linear infinite;
}

@keyframes flow {
  from { background-position: 0% 50%; }
  to   { background-position: 300% 50%; }
}
```

`background-size: 300%` 让渐变比元素宽，所以滚动 `background-position` 会显示不同的颜色段。最后一个颜色点和第一个相同，形成无缝循环。

对于像色轮一样旋转的渐变，注册一个角度属性，让浏览器可以给锥形渐变的起始角度做动画：

```css
@property --ang { syntax: '<angle>'; inherits: false; initial-value: 0deg; }

.conic-text {
  background-image: conic-gradient(from var(--ang), #58a6ff, #56d364, #d2a8ff, #ffa657, #58a6ff);
  animation: spin 4s linear infinite;
}

@keyframes spin { to { --ang: 360deg; } }
```

对于实时的 JavaScript 控制（例如用鼠标位置驱动渐变角度）：

```js
el.style.backgroundImage = `linear-gradient(${angle}deg, ${c1}, ${c2})`;
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 渐变 | 流动 | 流动是把颜色横向滑动，旋转是让它们像色轮一样转，斜向是滑动倾斜的色带，双色是在你选的两种颜色之间流动 |
| 速度 | 普通 | 一个循环需要的时间：慢速 6.4s，普通 4s，快速 2.4s；每个循环里，三倍于文字宽的渐变向左滑动两倍于自身的宽度，旋转的转一圈；慢一点感觉平静，快一点感觉活泼 |
| 第一种颜色 | 蓝色 | 用双色时，渐变开始和结束的颜色 |
| 第二种颜色 | 紫色 | 用双色时，中间的颜色 |
| 你的文字 | Flow | 颜色流过其中的文字 |

## 生产注意事项
- **`-webkit-background-clip: text`**：不带前缀的 `background-clip: text` 现在已经广泛支持（Chrome 119+、Firefox 122+、Safari 14+），但 `-webkit-` 前缀在所有版本的 Safari 上仍然需要。两个都写上。
- **必须用 `color: transparent`**：`background-clip: text` 把背景裁剪到文字形状，但文字颜色仍然会渲染在上面。没有 `color: transparent`，渐变会被藏在实色文字下面。
- **性能**：`background-position` 动画不用 CSS 合成器，每一帧都会触发绘制。对于长时间运行在大文字上的动画，先在中端设备上测性能。`@keyframes`（而不是 JS 的 `requestAnimationFrame`）可以避开主线程。
- **降级**：在渐变属性之前，先给元素设一个实色 `color`。不支持 `background-clip: text` 的浏览器会显示实色文字，而不是看不见的文字。
- **选中颜色**：在有些浏览器里，`color: transparent` 的选中文字可能没有可见的选中高亮。测一下选中行为，必要时加上 `::selection { color: white; background: blue; }`。
- **不需要库**：`background-clip: text` 和一个关键帧动画就是纯 CSS；当扫动必须和其他动画保持同步时，GSAP 可以补间 `backgroundPosition`。

## 另见
- [可变字体变形](../variable-font-morph/) — 字母本身变字重、变倾斜
- [描边填充](../outline-to-fill/) — 空心字填满颜色
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
