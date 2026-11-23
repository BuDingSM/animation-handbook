# 吸顶区域

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/sticky-section/)

## 它是什么
吸顶区域在你滚动时让整个区域固定不动，并用这段滚动让它的内容逐步走过一个序列，而不是移动页面。区域坐在一个高包装器里，你在包装器里滚了多远决定显示哪一步，所以往上滚会倒退回上一步。演示里，一列文字和一张图片一起走过四个步骤，然后区域滚走。

## 什么时候用
- 多步骤讲解，一个画面应该固定不动、内容向前推进
- 功能导览，插图和文案互换而视口不动
- 产品页面，“按住”主视觉，同时走过它的各个状态
- 任何固定画面比滚入新区域更能让读者保持方向感的序列

## 工作原理
包装器有四个舞台高；内部区域是 `position: sticky; top: 0`，所以包装器滚过时它保持固定。进度是 `(scrollTop − pinStart) / (pinHeight − stageHeight)`，钳制在 0–1 之间。乘以状态数量得到当前状态的序号：

```js
const pinStart = wrap.offsetTop, pinH = wrap.offsetHeight;
const p = clamp((st - pinStart) / (pinH - stageH), 0, 1);      // 0..1 while pinned
const si = Math.min(Math.floor(p * 4), 3);                  // state index

states.forEach((s, i) => s.classList.toggle('visible', i === si));
illus.forEach((il, i) => il.classList.toggle('visible', i === si));
```

状态通过 `.visible` 上的 CSS `opacity` 过渡交叉淡入淡出，所以区域读起来像固定画面在向前推进，而不是连续的 scrub。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 步骤数量 | 4 | 区域固定不动时，每步占用相等的滚动份额 |
| 包装器高度 | 四个盒子高 | 其中三个区域固定不动；包装器越高，每步持续越久 |
| 步骤之间的淡入淡出 | 0.5 s | 文字淡入下一步需要多久；图片需要 0.6 秒 |

## 生产注意事项
- **固定的是区域，不是一个元素**：这是与固定动画的关键区别。在这里，整个区域吸住，它的*内容*变化；而固定动画固定一个元素，让兄弟元素从它旁边滚过。
- **高度决定预算**：固定持续的滚动量完全由包装器的高度决定。想要更长的序列？把包装器做高——不需要改 JS。
- **吸顶胜过 JS 固定**：`position: sticky` 由浏览器的合成器处理，不会像滚动处理器里切换 `position: fixed` 那样抖动。把 JS 留给读取进度，而不是固定元素。
- **状态边界是阈值，不是事件**：因为状态序号来自 `floor(p × count)`，越过边界只是分数翻过去——即使丢了一帧滚动，也没有离散事件会错过。
- **减弱动态效果**：在 `prefers-reduced-motion` 下，状态和插图的过渡被禁用；内容仍然切换，只是直接切换而不是淡入淡出。
- **库的等价实现**：GSAP ScrollTrigger 用 `pin: true` 加一个 `scrub` 时间线是生产级版本，还能在状态之间加缓动；Framer Motion 把吸顶包装器与 `useScroll({ offset })` 配对，推导同样的进度值。

## 另见
- [固定动画](../pin-animation/) — 一个元素固定不动，其余部分从它旁边滚过
- [滚动 scrub 动画](../scrub-animation/) — 滚动让一架飞机沿路径来回移动
- [滚动叙事](../scrollytelling/) — 文字旁边的图片随故事滚动而变化
- [区域擦除切换](../section-wipe/) — 每个区域从上一个上面滑过
