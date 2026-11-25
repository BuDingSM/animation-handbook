# 滚动显现

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/reveal-on-scroll/)

## 它是什么
滚动显现让元素保持隐藏，比如淡出、移位、缩小、模糊或裁剪，直到它升过视野里的一条线，再让它落定到位。浏览器会在每个元素越过线时报告，所以每次滚动都不需要运行任何东西。演示用一条虚线展示了七种显现方式。

## 什么时候用
- 长页面滚动时，逐步披露内容区域、卡片或媒体
- 某个区块首次出现时吸引视线
- 每个区域都应该感觉经过精心安排、而不是预先渲染好的落地页
- 任何用轻量显现比用 JavaScript 跑滚动计算更合适的列表

## 工作原理
每张卡片带一个 `data-fx` 属性，写明它的技巧。CSS 按技巧定义隐藏状态，再定义一个共用的 `.revealed` 状态；观察者只切换这个类。触发位置变成底边的一个负 `rootMargin`，把观察者的边界拉上来，对齐虚线：

```js
const margin = -(100 - threshold);          // threshold 70 → -30
observer = new IntersectionObserver(entries => {
  entries.forEach(e => {
    const card = e.target.closest('.card');
    if (e.isIntersecting) {
      reveal(card);
      if (!repeatTog.checked) observer.unobserve(e.target);   // fire once
    } else if (repeatTog.checked) {
      hide(card);                                              // re-arm on exit
    }
  });
}, { root: scroller, rootMargin: `9999px 0px ${margin}% 0px`, threshold: 0 });
```

预设状态完全在 CSS 里——例如 `[data-fx="blur"]{opacity:0;filter:blur(12px)}` 变成 `.revealed[data-fx="blur"]{opacity:1;filter:blur(0)}`。`stagger` 技巧也不例外：它的子芯片按限定在 `.revealed` 下的 `transition-delay` 规则依次出现，所以光是这个类就驱动了整个组。观察者监听每张卡片本身，只有交错组例外，它监听的是那一排芯片：芯片在卡片里的位置更低，所以监听卡片顶部会在较低的触发线下、盒子外面就启动级联。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 触发线 | 普通 | 虚线所在的位置：更高是盒子一半处，普通是 70% 处，更低是 90% 处；线越高，卡片出现前要滚动得越远 |
| 每次都播放 | 关 | 关时每张卡片只出现一次并停住；开时它掉回线下会再次藏起，下次再播放 |

## 生产注意事项
- **一个观察者，多个元素**：一个 `IntersectionObserver` 处理所有卡片，比每次重新计算位置的 `scroll` 监听器便宜得多。观察者在主线程之外运行。
- **只触发一次还是重复播放**：第一次相交后调用 `unobserve` 是常见的生产选择——它能防止往上滚动时再次播放，并释放元素。只有真正想要重复效果时才继续观察。
- **`rootMargin` 的单位**：百分比是相对于根元素的尺寸。负的底边距是实现“元素升到视口 N% 处时触发”的标准技巧。
- **顶边不要设界。**顶边距为 `0px` 时，已经滚到触发线上方的元素会报告为*不*相交，所以快速一划或跳到滚动器末尾会直接跨过它：它永远不会被报告为进入，并一直以 `opacity: 0` 藏着。顶边距给一个大的正值，会让“在线上或线以上”始终成立，退出条件就只剩你真正想要的那一个——掉回线下。
- **触发标记不能放在滚动器里面。**滚动容器里绝对定位的子元素会随内容一起滚走，而观察者的触发点固定在视口上。把标记放在容器外面的非滚动包装器里——不然一滚动参考线就漂移，显现看起来会在任意位置触发。
- **让类驱动级联，别用定时器。**用 `setTimeout` 让子元素交错出现，会安排比启动它的状态活得更久的工作：级联中途滚走，已排队的回调还是会触发，把已经藏起的元素里的子元素重新显示出来。`:nth-child` 上的 `transition-delay` 与类绑定，所以离开视口时能干净地收回。只在显现状态下应用延迟，否则隐藏时也会反向交错。
- **静态标记的任何事都不属于滚动处理器。**它的位置取决于阈值和容器的高度，滚动时这两者都不会变——只在阈值变化和尺寸变化时重新计算。
- **减弱动态效果**：演示在 `prefers-reduced-motion` 下禁用过渡，所以内容仍然出现——只是跳过了运动。永远不要把可见性建立在动画完成上。
- **等效的库**：GSAP ScrollTrigger 的 `toggleActions` 和 AOS（Animate On Scroll）封装了完全相同的模式；Framer Motion 的 `whileInView` 属性是 React 里的等效做法，底层用的也是 `IntersectionObserver`。

## 另见
- [交错显现](../stagger-reveal/) — 一组里的元素一个接一个出现
- [滚动触发动画](../scroll-trigger/) — 动画在设定的滚动点开始、跟随并固定
- [联系人列表飞入飞出](../fly-in-fly-out-contact-list/) — 行在靠近边缘时淡入淡出、滑入滑出
- [滚动叙事](../scrollytelling/) — 旁边的图片随你往下读而变化
