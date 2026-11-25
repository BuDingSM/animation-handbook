# 缩放过渡

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/zoom-transition/)

## 它是什么
缩放过渡在淡入淡出的同时缩放页面，所以切换看起来像是在纵深中移动，而不是横着滑过屏幕。方向本身带有含义：放大感觉像抵达或深入，缩小像后退，而穿过——新页面从更大处冲进来再落定——感觉像在向前行进。

## 什么时候用
- 从总览深入到详情，放大能强化「深入」的感觉
- 关闭或退出一个视图，缩小表示离开
- 英雄区或开场时刻，穿过能增加动势
- 弹窗和灯箱的出现，应该感觉像内容向前走来

## 工作原理
每个变体是一组四个缩放值——`[oldStart, oldEnd, newStart, newEnd]`。页面从「起始」缩放开始，新页面透明，然后两者同时把 `transform` 和 `opacity` 过渡到「结束」状态：

```js
const V={
  'zoom-in': [1, 0.9, 0.75, 1],   // new page grows from 0.75 → 1 as old shrinks away
  'zoom-out':[1, 1.2,  1,   1],    // old page enlarges past the frame and fades
  'pull':    [1, 0.7,  1.3, 1],    // new rushes in from 1.3, old recedes to 0.7
}[variant];

o.style.transition='none'; n.style.transition='none';
o.style.transform=`scale(${V[0]})`;
n.style.opacity='0'; n.style.transform=`scale(${V[2]})`; n.classList.add('active');
void n.offsetWidth;                                       // commit the start scales
o.style.transition=t; n.style.transition=t;              // t = `all ${dur}ms ${ease}`
o.style.opacity='0'; o.style.transform=`scale(${V[1]})`;
n.style.opacity='1'; n.style.transform=`scale(${V[3]})`;
```

因为进入和离开的缩放值因变体而异，同一条代码路径能做出三种不同的空间感受。不透明度的交叉淡入淡出与缩放并行，所以两个页面重叠时都不会出现硬边；读一下 `offsetWidth` 会让浏览器在过渡开始前先应用起始缩放。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 缩放方式 | 放大 | 放大让新页面从 75% 长大，旧页面缩小到 90%；缩小让旧页面在淡出时放大到超过画面（120%），新页面以完整尺寸淡入；穿过让旧页面缩小到 70%，同时新页面从 130% 落定 |
| 速度 | 普通 | 缩放和淡入淡出需要多长时间：慢是 900ms，普通 550ms，快 330ms |
| 感觉 | 平滑 | 平滑减速到停；弹性会长得比完整尺寸稍大一点再回落；轻柔先慢后快再慢 |

## 生产注意事项
- **用 `transform: scale` 做动画，不要用 `width`/`height`。**缩放是合成的、便宜的；给尺寸做动画会让页面每一帧都重新布局、掉帧。
- **有意识地设置 `transform-origin`。**演示从中心缩放；一次应该从被点击的卡片开始的缩放，需要把原点设在那个点上，否则生长会感觉与触发脱节。
- **缩放的同时淡入淡出。**没有不透明度交叉淡入淡出的缩放会在两个页面重叠处留下一条硬边；成对的淡入淡出藏住了接缝。
- **弹性缓动可能冲过缩放 1**，在舞台边缘短暂裁掉内容——把过冲控制得小一点，或加上内边距。
- **减弱动态效果**会一下子换页，没有淡入淡出也没有缩放。
- **等价的库**：View Transitions API 可以用 `::view-transition-old`/`-new` 上的 `scale` 关键帧做缩放（View Transitions API 演示里的缩放样式）。Framer Motion 在 `AnimatePresence` 里的 `scale` 变体和 GSAP 的 `scale` 补间表达了同样的缩放加淡入淡出。

## 另见
- [View Transitions API](../view-transitions-api/) — 浏览器也能缩放页面
- [传送门 / 隧道缩放](../portal-zoom/) — 下一页从点击的圆圈里打开
- [滑动过渡](../slide-transition/) — 页面横向移动而不是缩放
- [共享元素过渡](../shared-element-transition/) — 一张图片长进下一页
