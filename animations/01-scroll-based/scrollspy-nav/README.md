# 滚动侦测导航

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/scrollspy-nav/)

## 它是什么

滚动侦测是长页面旁边的一份菜单，始终高亮你正在读的区域。区域标题滚到顶部附近的一条线以上时，它的链接亮起，标记滑向它；点击链接会滚动到那个区域。菜单和页面双向保持同步。

## 什么时候用

- 侧栏带目录的文档页面
- 带“本页内容”大纲的长文或法律页面
- 带锚点顶栏菜单的单页营销网站
- 分成带标签子区域的设置或个人资料页面
- 任何长到读者需要知道自己在哪里的页面

## 工作原理

每个区域的顶部只测量一次，缓存在滚动容器自己的坐标空间里，然后每次滚动帧都把一条*激活线*——滚动偏移加上一部分视口高度——与这些缓存的顶部比较。激活的区域是顶部在该线之上的最后一个。只有激活的索引真正变化时才碰 DOM，指示器靠 CSS 的 `transform` 过渡移动，所以一次状态变化只花一次样式写入。

```js
function measure() {
  const sRect = doc.getBoundingClientRect();
  tops = sections.map(el => {
    const r = el.getBoundingClientRect();
    return r.top - sRect.top + doc.scrollTop; // container-relative, not offsetTop
  });
  maxScroll = doc.scrollHeight - doc.clientHeight;
}

function spy() {
  if (targetLock > -1) return;            // programmatic scroll in flight
  const line = doc.scrollTop + doc.clientHeight * activation;
  let i = 0;
  for (let k = 0; k < tops.length; k++) if (tops[k] <= line) i = k;
  if (doc.scrollTop >= maxScroll - 1) i = tops.length - 1;
  if (i !== active) setActive(i);         // guarded swap — one write per change
}
```

点击链接会把 `targetLock` 设为目标索引，立刻激活它，并开始平滑滚动：每帧设置 `scrollTop` 的短暂缓动；如果“滑向该区域”关了、或者访客开启了减弱动态效果，就直接跳过去。锁住期间，`spy()` 跳过自己的计算，指示器就不会在滑过每个中间区域时闪烁。滑到目的地时锁解除，访客用滚轮、触摸或按键接管、或者按下播放或回到顶部时也立刻解除。不用 `behavior: 'smooth'`、而由页面自己滑，是它能干净结束的原因：在 Chrome 里，浏览器驱动的平滑滚动可能在瞬间跳跃落下时还在运行，盒子就会在顶部前一点点停住。

菜单在滚动器里面，位于一个零高、`position: sticky; top: 0` 的行里，所以区域移动时它保持在原位，从它上面开始的滚轮转动或滑动仍然滚动盒子。`setActive()` 还会给当前链接打上 `aria-current`。菜单自己滚动时（手机上顶部的一条横条、横屏短屏幕上的高菜单），它设置菜单自己的滚动位置来显示那个链接；`scrollIntoView()` 会连页面一起滚。一帧内跳过一盒半以上高度（回到顶部、从头重新播放）会立刻重置标记，而不是让它滑过去。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 链接切换点 | 三分之一处 | 区域标题必须到达多高，它的链接才会点亮：靠近顶部是盒子 20% 处，三分之一处是 35%，一半处是 50%；线越高，切换越晚 |
| 滑向该区域 | 开 | 链接滑向它的区域；关时立刻跳过去，减弱动态效果下也总是这样 |

## 生产注意事项

- **导航不能随内容一起滚走。**滚动容器里绝对定位的元素会随内容一起滚走。把轨道放在容器外面的非滚动包装器里（滚动器就是页面本身时用 `position: sticky` / `fixed`）。这个演示反过来做：菜单放在一个零高、`position: sticky; top: 0` 的行里，它是滚动器的第一个子元素，所以它保持不动，从菜单上开始的滚轮转动或滑动会滚动盒子，而盒子旁边的菜单会滚动页面。
- **`offsetTop` 是反复出现的陷阱。**`offsetTop` 是相对于最近的定位祖先，在嵌套滚动器布局里通常*不是*滚动容器——那时每个比较都错了（这个演示定位了自己的滚动器，但仍然按下面的方法测量，不管中间有什么都管用）。用 `getBoundingClientRect()` 相对于容器的矩形加上它的 `scrollTop` 来测量，尺寸变化时重新测量。
- **最后一个区域常常太短，激活不了。**如果它比激活线到视口底部的距离还短，它的顶部永远越不过那条线。`scrollTop` 到达 `maxScroll` 时强制设为最后一个索引（这个演示还加了一段尾部空间，让最后一个区域有地方）。没有这个，最后一个导航链接靠滚动永远到不了。
- **编程式滚动期间压制侦测更新。**从区域 1 平滑滚动到区域 5 会经过 2、3、4；没有锁，指示器会快速扫过中间的每个链接。立刻激活目标，滚动稳定之前忽略侦测结果，然后恢复。
- **IntersectionObserver 不是直接替代品。**它告诉你哪些区域与一个带相交，但滚动侦测在任何时候都需要*恰好一个*激活区域——比视口还高的区域与细带没有交集，除非你用精心调过的 `rootMargin` 带观察，两个相交区域之间的平局仍然需要滚动位置来决胜负。对于“一个激活链接”的语义，缓存顶部比较更简单、更确定。
- **改用原生锚点？**如果你用 `href="#section"` 链接、让浏览器跳过去，给目标设 `scroll-margin-top`，让固定顶栏不遮住区域标题，并在滚动器上用 `scroll-behavior: smooth`。
- **库**：Bootstrap 的 Scrollspy 在滚动容器有 `data-bs-spy="scroll"` 和一个指明菜单的 `data-bs-target` 后，会高亮视野中区域的链接；Gumshoe 是做同样事的小型独立脚本。

## 另见

- [滚动触发动画](../scroll-trigger/) — 页面各部分滚过时，东西开始、scrub 和固定
- [阅读进度条](../progress-bar/) — 一条横条显示你读了多远
- [滚动吸附](../snap-scrolling/) — 滚动一次停在一个完整区域上
