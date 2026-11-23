# 弹跳进入

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/bounce-in/)

## 它是什么

弹跳进入把元素从小而低的位置带上来，超过它的最终大小和位置，再弹回静止，就像带着动量落地的东西。真正的弹跳不是一条平滑的曲线：它是几个手工放置的步骤，每一步都稍稍超过目标再回来。

## 什么时候用
- 好玩的确认——成功徽章、「消息已发送」对勾、已加入购物车标签
- 轻提示、提示框和弹出层，一点点生气让出现感觉很灵敏
- 游戏界面、儿童产品和语气欢迎个性的品牌
- 任何小到一次精力充沛的入场不会显得沉重的单个元素

## 工作原理
卡片通过一个命名的关键帧而不是过渡来做动画。每个停靠点都缩放到 1 以上再拉回 1 以下，同时平移垂直的超过。超过的量是根据「弹跳力度」选项（强度 30、60 或 100）算出来的，这样路径点保持成比例：

```js
const t=intensity/100;
const ov1=1+t*0.15,un1=1-t*0.07;   // overshoot, then undershoot
kf=`@keyframes bounce-in{
  0%  {opacity:${fadeIn?0:1};transform:scale(0.3) translateY(40px)}
  55% {opacity:1;transform:scale(${ov1.toFixed(3)}) translateY(-8px)}
  80% {transform:scale(${un1.toFixed(3)}) translateY(3px)}
  100%{opacity:1;transform:scale(1) translateY(0)}
}`;
```

```css
.card{opacity:0;transform:scale(0.3) translateY(40px)}
.card.in{animation:bounce-in var(--dur) ease forwards}
```

生成的 CSS 被注入一个实时的 `<style>` 标签，所以改「弹跳力度」或「弹跳次数」会当场重写关键帧规则。两次和三次弹跳模式只是在更紧的百分比间隔里加更多的超过/不足对，每次都比上一次小，模仿能量耗散。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 弹跳力度 | 中等 | 长到比完整尺寸大多少：「轻」大约 5%，「中等」9%，「强」15%，再缩回下面一点，大约比完整尺寸小 2%、4% 和 7% |
| 弹跳次数 | 一次 | 超过多少次；几乎总是一次，因为多了会显得卡通 |
| 速度 | 普通 | 要多长时间：「慢」是 1800ms，「普通」1100ms，「快」650ms；太短了，弹簧读起来像抖动 |
| 淡入 | 开 | 弹跳时是否淡入，还是一开始就可见 |

## 生产注意事项
- **关键帧胜过缓动**：像 `cubic-bezier(.34,1.56,.64,1)` 这样的超过弹簧只给*一次*超过。真正的多阶段弹跳和挤压拉伸需要明确的路径点——这就是这个技术用 `@keyframes` 的全部原因。
- **一次弹跳通常是对的**：每多一个弹跳循环，读起来就更幼稚。两次和三次留给故意做得像玩具的场景。
- **`animation-fill-mode: forwards`**：没有它，元素会在动画结束后一下子回到 `scale(0.3)` 的起始状态；`forwards` 会保持最后一帧。
- **减弱动态**：在 `prefers-reduced-motion` 下，演示完全禁用关键帧动画，把元素显示在静止状态——弹跳是经典的晕动触发因素。
- **Framer Motion**：用带 `stiffness`/`damping` 的 `type:"spring"` 过渡，或者弹簧上的 `bounce`——物理模型不用写关键帧就能产生超过。
- **GSAP**：`Bounce.out` / `Elastic.out` 缓动，或者带超过值的 `.fromTo()`，不用手写关键帧块就能做到。

## 另见
- [缩放进入](../scale-in/) — 放大到位，没有弹跳
- [旋转进入](../rotate-in/) — 放大时旋转到位
- [滑入](../slide-in/) — 从一边滑入到位
