# 路径文字

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/text-on-path/)

## 它是什么
路径文字让一行字沿着曲线走，而不是直线。曲线定义一次，文字附在上面，所以每个字母都坐在那条线上。把文字开始的点沿着曲线移动，字母就会沿着它行进，就像一条弯成形状的滚动条。

## 什么时候用
- 圆形徽章、印章和图章，文字绕着一个环
- 弧形、波浪或螺旋形的装饰标题，横跨主视觉
- 图表和信息图，标签应该贴着路线或连接线
- 沿着形状走的循环跑马灯，而不是平直的线

## 工作原理
路径在 `<defs>` 里用一个 `id` 声明一次。一个 `<textPath href="#curve">` 引用它，字形自动沿着曲线排布：

```html
<defs><path id="curve" d="M20,180 C120,60 240,300 340,180 S560,60 620,180"/></defs>
<text class="flow-text">
  <textPath href="#curve" id="tp" startOffset="0%">FOLLOW THE CURVE · </textPath>
</text>
```

跑出路径末端的字形不会被画出来，所以如果只有一个词组副本，它滑过去时曲线的末端会空出来。替代做法是把词组重复到盖住整条路径再加一个副本。用两个副本减一个副本来量一个副本的长度（用 `getSubStringLength()`），这样两端会塌掉的空格就抵消了。起点只在一个副本的长度内移动，从路径起点之前一个副本的位置移到路径起点。包住时，一个一模一样的副本已经在原位，所以跳变看不出来：

```js
// unit: the length of one copy of the phrase, as a percentage of the path
tp.textContent = phrase.repeat(Math.ceil(100 / unit) + 1);    // the whole path plus one extra copy

function loop(now) {
  const dt = last === null ? 0 : Math.min(now - last, 50);    // milliseconds since the last frame
  last = now;
  offset = (offset + speed * dt / (1000 / 60)) % unit;        // speed: % of the path per 1/60 s; wraps after one copy
  tp.setAttribute('startOffset', (offset - unit) + '%');      // always between -unit and 0
  requestAnimationFrame(loop);
}
```

移动跟随的是时钟，而不是帧数，所以 120 Hz 的屏幕并不比 60 Hz 的快。开始后的第一帧只记录时间，帧之间的长间隔最多按 50 ms 算，所以暂停后重新开始不会跳跃。

切换路径形状会换同一个 `<path>` 上的 `d` 属性，输入新词组会替换文字。无论哪种情况，一个副本现在占曲线的比例都不同了，所以要重新测量词组并重复，起点也马上写好，保持在新的副本长度内。这样即使动画暂停，整条曲线也仍然被盖住。虚线引导是同一条路径的 `<use href="#curve">`，用透明度开关。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 路径形状 | 波浪 | 文字跟随的曲线：波浪、弧线或完整的圆 |
| 速度 | 普通 | 文字每六十分之一秒移动多远：慢速是路径的 0.25%，普通 0.4%，快速 0.65%（每秒 15%、24% 和 39%） |
| 显示路径 | 开 | 把曲线画成文字下面的虚线 |
| 你的文字 | FOLLOW THE CURVE · | 会行进的词组；结尾加一个分隔符，这样重复读起来才顺 |

## 生产注意事项
- **无障碍**：SVG 文字仍然是真实的、可选的文字，按 DOM 顺序朗读——在 `<svg>` 上加一个描述词组的 `aria-label`，因为动画偏移可能在视觉上把一个词在包住点处拆开。
- **紧曲线上的可读性**：急转弯会让字形挤在一起或散开。减小 `letter-spacing`、缩短字符串，或者缓和曲率。`textPath` 在拐角周围没有自动的字距修正。
- **减弱动态效果**：把 `requestAnimationFrame` 循环放在 `prefers-reduced-motion` 检查后面，在一个固定的 `startOffset` 渲染文字，这样不用滚动也能保持可读。
- **库的等价做法**：GSAP 的 MotionPathPlugin 让元素（不只是文字）沿着路径运动，还能自动旋转；D3 提供了方便数据驱动曲线的 `path` 生成器。专门做圆形文字时，有 CSS `writing-mode` 的技巧，但 `<textPath>` 仍然是最灵活的。

## 另见
- [跑马灯/滚动条](../marquee-ticker/) — 文字沿直线滚动
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
