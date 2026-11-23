# 跑马灯/滚动条

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/marquee-ticker/)

## 它是什么
跑马灯让一行文字横向无缝滚动，就像新闻滚动条。文字是重复的，整行移动的距离正好是一个副本的宽度，然后再从头开始，所以总有一个一模一样的副本在原位，接缝看不出来。在演示里，三行以同样的速度滚动，中间那行方向相反，指向某一行或点一下它，只会停住那一行。

## 什么时候用
- 滚动的新闻滚动条和体育比分
- 机构/作品集网站上的品牌内容轨道（“CLIENT LIST · WORK · PROCESS ·”）
- 无限滚动的 Logo 条（滚动的客户 Logo）
- 主视觉里的氛围背景文字
- 两行反向跑马灯，作为一种装饰图案

## 工作原理
轨道里放着内容，精确地复制了两份。CSS 动画把轨道平移总宽度的 `-50%`——也就是一个副本的宽度。在循环点上，第二个副本正好在第一个副本开始的位置，所以过渡看不出来：

```html
<div class="overflow-container">
  <div class="track">
    <span>Content · Content · Content ·</span>
    <span>Content · Content · Content ·</span>  <!-- duplicate -->
  </div>
</div>
```

```css
.overflow-container { overflow: hidden; }

.track {
  display: flex;
  width: max-content;
  white-space: nowrap;
  animation: marquee 10s linear infinite;
}

@keyframes marquee {
  from { transform: translateX(0); }
  to   { transform: translateX(-50%); }
}
```

要按目标每秒像素数算出正确的时长：

```js
// The animation travels exactly one copy's width (50% of the full track)
const oneCopyWidth = trackEl.scrollWidth / 2;
const durationSeconds = oneCopyWidth / pixelsPerSecond;
trackEl.style.animationDuration = durationSeconds + 's';
```

悬停时暂停：

```css
.overflow-container:hover .track {
  animation-play-state: paused;
}
```

反向是把动画从 `-50%` 做到 `0`：

```css
@keyframes marquee-reverse {
  from { transform: translateX(-50%); }
  to   { transform: translateX(0); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 文字移动的速度：慢速每秒 50px，普通 80px，快速 130px；新闻滚动条跑得慢一点，品牌条跑得快一点 |
| 文字大小 | 中 | 小是 32px，中 48px，大 64px；大字是品牌条的常用选择 |
| 项目间距 | 中 | 每处重复的文字和它分隔符周围的间距：紧凑是 24px，中 48px，宽松 80px |
| 分隔符 | 闪光 ✦ | 重复之间的标记：圆点、星星、破折号或闪光 |
| 你的文字 | 动画手册 | 滚动的文字 |

## 生产注意事项
- **两个副本，不要更多**：只要一个副本比视口宽，两个副本对任何视口宽度都够了。更多的副本浪费 DOM。如果你的文字很短，在每个副本里多加几次重复，而不是加更多副本。
- **轨道上用 `width: max-content`**：可以防止换行。没有它，文字会在容器宽度处换行，破坏布局。
- **轨道元素上的 `will-change: transform`**：会把它提升到 GPU 合成层。加之前先测性能——它会占用 GPU 内存，在低端手机上可能出问题。
- **悬停暂停对体验至关重要**：想读具体内容的用户需要能暂停。`:hover` 的 CSS 做法是零 JavaScript。
- **无障碍**：给跑马灯容器加 `aria-live="off"`——屏幕阅读器不应该不停地播报滚动内容。如果内容里有链接，要确保键盘可以到达（焦点状态可见，tabIndex 正确）。
- **减弱动态效果**：`@media (prefers-reduced-motion: reduce)` → `animation-play-state: paused`。内容仍然可见，只是不动了。
- **Swiper.js** 有 `autoplay` + `loop` 模式，处理跑马灯行为，触摸支持和无障碍都内置了。

## 另见
- [轮换词轮播](../rotate-word-carousel/) — 句子中的一个词不断换成下一个
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [路径文字](../text-on-path/) — 文字沿曲线滚动，而不是直线
