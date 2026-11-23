# 数字滚动计数

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/counter-animation/)

## 它是什么

数字滚动计数动画会在数字滚动进入视野时，把它们从零数到总数，让一排统计数据看起来像是到场了，而不只是摆在那里。每个数字在整个上升过程中都保留逗号、货币符号或单位，接近总数时通常会慢下来，稳稳落定。

## 什么时候用

- 需要第一眼就用视觉强调突出原始数字的统计与指标区
- 展示关键数据的年度报告、仪表盘和 SaaS 落地页
- 任何“数字落定”能传达“到场”而不是静态展示的场合
- 数据逐步到位时计数器随之填满的加载过程

## 工作原理

IntersectionObserver 监听统计容器，在它越过内部滚动视口里可配置的阈值时触发：

```js
const margin = -(100 - threshold); // e.g. threshold 60 → margin -40
observer = new IntersectionObserver(entries => {
  if (entries[entries.length - 1].isIntersecting && !running) animateCounters();
}, { root: stage, rootMargin: `0px 0px ${margin}% 0px`, threshold: 0 });
observer.observe(document.getElementById('stats'));
```

`root: stage` 很关键——舞台是内部滚动的。没有它，观察器默认以文档视口为准，而统计元素在页面层面一直“可见”，所以永远不会触发。

rAF 循环用经过的时间和缓动函数从 0 插值到目标值：

```js
function frame(now) {
  const t = Math.min((now - begun) / duration, 1);   // begun: when this number's own count started
  const v = target * easeFn(t);
  el.textContent = format(v);          // formatted every frame
  if (t < 1) requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```

格式化发生在每一帧——而不只在最后。这样可以保证逗号、货币符号和单位在整个动画过程中都出现，而不是在最终值处突然跳出来。

每次计数都从零开始。观察器决定数字何时可以开始；然后每个数字在前一个开始之后（加上级联间隔）并且自身一半进入盒子后才开始，这样即使统计容器越过触发线时下面一行的数字还在盒子外面，滚动进来时也能看到它在计数。速度、曲线和级联都在计数开始时读取，所以改动从下一次计数开始生效。开启减弱动态效果后，数字直接跳到总数。

收入数字在变大过程中会换格式，从美元到千美元再到百万美元，所以它最宽的字符串永远不会比最终的“12,847”更宽，在手机上也始终放得进卡片。

第二个观察器在底部缩小了 8px，监听统计容器完全离开盒子的时刻；它会停掉正在进行的计数并把数字归零，这样回到顶部或播放到结尾后再滚进来时，数字从零开始，而不是先显示旧的总数。它的边距比任何一条开始计数的线都小，所以计数观察器总是先松手，往下滚时可以重新开始计数。

四种“感觉”选项用这些曲线：均匀是 `linear`，平滑是 `outCubic`，慢收尾是 `outExpo`，有弹性是 `outBack`：

```js
const EASE = {
  linear:   t => t,
  outCubic: t => 1 - (1 - t) ** 3,
  outExpo:  t => t === 1 ? 1 : 1 - Math.pow(2, -10 * t),
  outBack:  t => { const c = 1.70158; return 1 + (c+1)*Math.pow(t-1,3) + c*Math.pow(t-1,2) }
};
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 开始计数 | 中 | 卡片顶部必须越过、数字才能开始计数的线：早是盒子向上十分之一处，中是向上 40% 处，晚是 70% 处。每个数字还要等自身一半进入盒子，所以在矮盒子里早和中几乎从同一个地方开始 |
| 速度 | 普通 | 每次计数的时间：慢是 2.9 秒，普通 1.8 秒，快 1.1 秒 |
| 感觉 | 平滑 | 平滑会减速停下；慢收尾先冲到接近总数，再慢慢走完最后几位；有弹性会稍微超过再落回来；均匀以一个恒定速度计数，感觉很机械 |
| 一个接一个 | 开 | 每个数字比前一个晚 200ms 开始，所以四个数字级联计数 |

## 生产注意事项

- **`root` 必须指向内部滚动容器，而不是文档。** 如果统计区在自定义滚动容器里，`root: null`（默认值）会以页面视口为准观察——该区域在页面上立即“可见”，观察器在真正滚动时永远不会触发。一定要传 `root: stage`（或任何带 `overflow-y: scroll` 的元素）。
- **`rootMargin` 的百分比是相对于根元素的边框盒。** 用 `root: stage` 时，`-40%` 指舞台高度的 40%，而不是窗口。这让阈值位置与窗口大小无关，保持一致。
- **每一帧都要格式化，而不只在最后。** 只在完成时格式化，会导致从原始小数到格式化字符串的突兀跳变。从 0 数到 12,847 的计数器，应该在第一次越过 1,000 时就显示逗号。
- **`easeOutBack` 会超调。** 数值在落定前会短暂超过目标值。对计数器来说，显示的数字可能有一两帧高于目标值。目标值在视觉上绝不能被超过时，用 `outCubic` 或 `outExpo`。
- **GSAP 等价写法：** `gsap.to(obj, { val: target, duration: 1.8, ease: "power3.out", onUpdate: () => el.textContent = format(obj.val) })`。CountUp.js 是专门做这种模式的库，内置了本地化格式化。

## 另见

- [滚动显现](../reveal-on-scroll/) — 卡片滚动进入视野时出现
- [交错显现](../stagger-reveal/) — 一组里的元素随滚动依次出现
- [数字滚动](../../04-micro-interactions/rolling-numbers/) — 每个数字滚动到新值，而不是从零数起
