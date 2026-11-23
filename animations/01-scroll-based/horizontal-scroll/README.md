# 横向滚动

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/horizontal-scroll/)

## 它是什么

横向滚动把普通的向下滚动变成横向移动。一排面板坐在一个高区域里的长条中；滚动经过这个区域时，它保持不动，长条向左滑动，一个接一个面板，然后页面继续往下。你永远不用自己横向滚动。

## 什么时候用

- 作品集项目展示，每个面板都是一件独立的代表作
- 功能导览，横向移动强化了“穿行其中”的叙事
- 时间线、流程或编号步骤，适合从左到右的阅读顺序
- 任何想实现横向导航、又不想让用户横向滚动或用方向键的场合

## 工作原理

两个公式完成全部工作——固定距离和 translateX 偏移。

**固定区域高度**决定了有多少垂直滚动预算：

```css
.stage { container-type: size; }                /* 100cqh is one stage height */
#pin-section { height: calc(100cqh * var(--panels)); }
/* 5 panels × a 380px stage = 1900px tall section */
```

**轨道尺寸**——轨道必须明确比容器宽，每个面板必须参照轨道的宽度，而不是舞台的宽度：

```css
#h-track { display: flex; width: calc(var(--panels) * 100%); }
/* 100% here = #pin-inner = stage width; 5 × 100% = 5× stage width */

.panel  { flex: 0 0 calc(100% / var(--panels)); }
/* 100% of track ÷ 5 = exactly one stage-width per panel */
```

**随滚动平移：**

```js
const budget   = pinSection.offsetHeight - stage.clientHeight;
const progress = clamp((stage.scrollTop - pinSection.offsetTop) / budget, 0, 1);
const maxShift = track.offsetWidth - stage.clientWidth;
track.style.transform = `translateX(${-progress * maxShift}px)`;
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 面板数量 | 5 | 每个面板一个盒子宽，长条和它们加起来一样宽 |
| 固定区域高度 | 五个盒子高 | 盒子滚过它时，长条滑动；更高的区域让每个面板经过得更慢 |
| 前导和尾随 | 各六成盒子高 | 区域前后的空间，让你看到它到来、停住再松开 |

## 生产注意事项

- **轨道必须有明确的宽度。** 没有 `width: calc(var(--panels) * 100%)`，轨道的 `offsetWidth` 就等于包含块的宽度。于是 `maxShift = track.offsetWidth - stage.clientWidth = 0`，`translateX` 永远是零——轨道永远不动。这是这种模式最常见的失败原因。
- **面板的 flex-basis 必须参照轨道宽度，而不是舞台宽度。** `flex: 0 0 100%` 里 100% 解析为容器（轨道，而不是舞台），只有轨道有了明确宽度后才正确。如果轨道没有明确宽度，flex-basis 也会塌掉。
- **滚动容器和吸顶内层之间不要有 `overflow: hidden`。** 如果吸顶元素和滚动容器之间的任何祖先把 `overflow` 设成 `visible` 以外的值，`position: sticky` 就失效了。`#pin-inner` 在 `#pin-section` 里吸顶，`#pin-section` 在 `.stage` 里——中间的元素都不应该有 `overflow: hidden`。
- **滚动容器需要 `position: relative`。** `pinSection.offsetTop` 是相对于最近的*定位*祖先测量的，而不是最近的滚动祖先。如果滚动容器是静态定位，`offsetTop` 会悄悄地以 `<body>` 为准算出，包含页面顶栏，于是 `progress` 开始得晚，还没等固定松开就饱和到 1——最后一个面板永远到不了。
- **在纵向 flex 布局里，`flex: 1` 会覆盖 `height`。** 手机上侧边面板堆在滚动容器下面时，`flex: 1 1 0%` 让舞台的 flex-basis 成为主尺寸，于是固定的 `height` 被忽略，舞台长到内容的完整高度。它不再是滚动容器，`scrollTop` 永远是 0，整个效果就死了。在手机媒体查询里把它重置为 `flex: none`。
- **`progress` 必须用 `stage.scrollTop` 算，而不是 `window.scrollY`。** 滚动容器是内部的舞台元素。用 `window.scrollY` 永远返回 0。
- **GSAP ScrollTrigger 等价写法：** `ScrollTrigger.create({ trigger, start, end, scrub: true, onUpdate: self => track.style.transform = 'translateX(...)' })`。横向滚动区域在 Locomotive Scroll 和 Lenis 里也是一等公民功能。
- **移动端考虑。** 固定区域有五个舞台高：300px 的手机舞台上就是 1,500px。在触摸设备上测试一下这种感觉是否自然——垂直滚动与横向移动的比例决定了感知到的“阻力”。移动得快（每像素面板少）感觉轻；移动得慢（面板多）会感觉粘滞。

## 另见

- [吸顶区域](../sticky-section/) — 区域固定不动，内容在里面变化
- [滚动 scrub 动画](../scrub-animation/) — 滚动让一架飞机沿路线移动，正反都行
- [滚动叙事](../scrollytelling/) — 故事滚过时，旁边的图片随之变化
