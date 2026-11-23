# 3D 翻转卡片

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/flip-card-3d/)

## 它是什么
3D 翻转卡片有正面和背面，在 3D 中翻过来露出另一面。两面处在同一个位置，背面转过去，背对观看者的那一面会被隐藏。卡片容器上的透视让翻转有了真正的景深，感觉就像翻一张真实的卡片。

## 什么时候用
- 个人、团队或联系卡片，背面放次要信息
- 背面显示规格或价格的产品卡片
- 问题和答案各在一面的闪卡和测验界面
- 悬停显示标题或行动号召的画廊瓷贴
- 任何直白的交叉淡入淡出显得太平、想要触感十足的实体翻转的地方

## 工作原理
三个属性完成这件事。`perspective` 放在**容器**上，`preserve-3d` 放在旋转的**内部**元素上，`backface-visibility: hidden` 放在**每一面**上。背面预先旋转 180 度，卡片翻过来后读起来才是正的：

```css
.stage { perspective: 1200px; }            /* camera depth, on the parent */

.card {
  transform-style: preserve-3d;            /* faces share one 3D scene */
  transition: transform 650ms cubic-bezier(.6,.02,.2,1);
}
.card.flipped { transform: rotateY(180deg); }

.face {
  position: absolute; inset: 0;
  backface-visibility: hidden;             /* hide the side facing away */
}
.back { transform: rotateY(180deg); }      /* pre-flip so it faces out */
```

切换只用一个 class，所以同样的标记能用于点击、键盘和悬停触发：

```js
card.addEventListener('click', () => card.classList.toggle('flipped'));
card.addEventListener('keydown', e => {
  if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); card.classList.toggle('flipped'); }
});
```

静止时的倾斜与翻转是分开的。透视容器和卡片之间的一个包装层会朝指针旋转几度，用 Pointer Events 跟踪，所以鼠标和手写笔都适用：

```js
stage.addEventListener('pointermove', e => {
  const r = stage.getBoundingClientRect();
  const x = (e.clientX - r.left) / r.width  - 0.5;
  const y = (e.clientY - r.top)  / r.height - 0.5;
  tilt.style.transform = `rotateX(${-y * 10}deg) rotateY(${x * 10}deg)`;
});
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 翻转方向 | 横向 | 横向像翻书一样翻转卡片；上下翻像翻盖一样 |
| 速度 | 普通 | 翻一次要多长时间：慢是 1000ms，普通 650ms，快是 400ms；600 到 900ms 感觉最像实体 |
| 3D 景深 | 中等 | 观看者看起来有多近：轻微是 2000px，中等 1200px，强烈 600px；越近翻转越夸张 |
| 悬停翻转 | 关 | 指向卡片会翻过去，离开会翻回来；触屏上轻触仍然能翻 |
| 朝指针倾斜 | 轻柔 | 静止时，卡片朝悬停的指针倾斜，在舞台边缘最多 5°（轻柔）或 9°（强烈）；关掉就保持平放 |

## 生产注意事项
- **backface-visibility 支持**：两面必须绝对定位并叠在一起，内部元素必须有 `preserve-3d`。如果缺少 `preserve-3d`（或 `overflow`、`filter` 之类的属性压平了上下文），两面都会渲染，背面会透出来。
- **Safari 闪烁**：WebKit 可能在旋转中途闪一下隐藏的那面。加上 `-webkit-backface-visibility: hidden`，并用 `transform: translateZ(0)`（或 `translateZ(1px)`）把每面推一下，强制形成稳定的合成层。`preserve-3d` 元素上不要用 `overflow`/`filter`，它们可能让 3D 上下文塌掉。
- **无障碍**：翻转必须能用键盘触发——让卡片可以获得焦点（`tabindex="0"`、`role="button"`），按 Enter/Space 翻转，并有可见的焦点环。因为 `backface-visibility: hidden` 只隐藏视觉，背面的文字仍在无障碍树里，可以到达；用 `aria-pressed` 暴露状态。不要只把背面内容藏在悬停后面。
- **减弱动态**：在 `prefers-reduced-motion: reduce` 下，完全去掉旋转和倾斜，改用快速的不透明度淡入淡出切换两面（演示的淡入淡出是 120ms），这样信息仍然可达，又不会引发前庭不适的运动。
- **库**：Framer Motion 用 motion 组件上的 `rotateY` 和 `AnimatePresence` 做换面来做这个动画；GSAP 用 `rotationY` 加 `transformPerspective`/`transformStyle` 来处理。上面纯 CSS 的版本在常见情况下不需要依赖。

## 另见
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜，不翻过来
- [翻转进入](../../02-entrance-and-exit/flip-in/) — 一张卡片在 3D 铰链上摆进视野
- [滚动驱动 3D 旋转](../scroll-driven-3d-rotation/) — 滚动转动一个 3D 立方体
- [翻页卷角](../../03-page-transitions/page-curl/) — 整页像纸一样翻过来
