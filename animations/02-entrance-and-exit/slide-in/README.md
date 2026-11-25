# 滑入

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/slide-in/)

## 它是什么

滑入把元素从不远处移到最终位置。移动有方向（从上、下、左或右），所以元素看起来像是从某个地方来的。配上淡入淡出，感觉像一次自然的入场；单独用可能会感觉很机械。

## 什么时候用
- 从屏幕边缘进入的面板、抽屉和底页
- 从角落滑入的轻提示和通知横幅
- 交错的列表或卡片入场，每一项都向上滑一小段
- 任何到达方向本身就有意义的内容（比如从顶部滑下的导航）

## 工作原理
卡片带有一个偏移 transform，还可以带 `opacity: 0`。偏移和淡入淡出都存在 CSS 变量里，由方向和距离控件改写。加上 `.in` 会把 transform 一下子变回 `translate(0,0)`、不透明度变为 1，`transition` 再把这个变化做成动画：

```css
.card {
  opacity: var(--fade);
  transform: translate(var(--tx), var(--ty));
  transition: transform var(--dur) var(--ease),
              opacity var(--dur) var(--ease);
  will-change: transform, opacity;
}
.card.in { opacity: 1 !important; transform: translate(0, 0) !important; }
```

JavaScript 只根据选好的方向和距离算出起始偏移：

```js
const tx = dir === 'left' ? -dist : dir === 'right' ? dist : 0;
const ty = dir === 'top'  ? -dist : dir === 'bottom' ? dist : 0;
root.style.setProperty('--tx', tx + 'px');
root.style.setProperty('--ty', ty + 'px');
root.style.setProperty('--fade', withFade ? '0' : '1');
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 从哪边进入 | 上 | 它从哪条边滑进来 |
| 移动多远 | 中等 | 开始时离得有多远：「短」是 50px，「中等」200px，「远」400px；短距离适合微妙的界面动作，长距离感觉很夸张 |
| 速度 | 普通 | 滑动要多长时间：「慢」是 1000ms，「普通」600ms，「快」350ms；距离越长，需要的时间越多，速度才显得可信 |
| 感觉 | 平滑 | 「平滑」慢慢停稳；「弹回」加一点点超过；「轻柔」两头放慢；「均匀」保持一个稳定的速度 |
| 淡入 | 开 | 单独的滑动看起来很机械；淡入淡出让它读起来像是一次到达 |

## 生产注意事项
- **用 `transform`，永远不要用 `top`/`left`/`margin`。**平移只走合成器；给布局属性做动画会每帧都强制重排，在手机上掉帧。
- **给容器裁剪**：元素从屏幕外滑入时，给容器加 `overflow: hidden`，否则动画前的偏移可能产生横向滚动条，或者溢出到预期的范围之外。
- **真实界面里移动要短。**营销主视觉的入场可以移动 100–200px，但界面元素只滑 8–24px 感觉更利落——足够暗示方向，又不用走很远。
- **减弱动态：**在 `prefers-reduced-motion` 下，演示去掉平移，只留一个短暂的不透明度淡入淡出，内容仍然出现，但不会飞过屏幕。
- **库里的等价写法：**GSAP 的 `gsap.from(el, { x: 200, autoAlpha: 0 })`；Framer Motion 的 `initial={{ x: 200, opacity: 0 }} animate={{ x: 0, opacity: 1 }}`；Motion One 的 `animate(el, { transform: ['translateX(200px)', 'none'], opacity: [0, 1] })`。

## 另见
- [淡入淡出](../fade-in-out/) — 单纯的淡入淡出，没有移动
- [上滑揭示](../slide-up-reveal/) — 文字从一条看不见的边缘后方升起
- [缩放进入](../scale-in/) — 放大到位，而不是移动
- [弹跳进入](../bounce-in/) — 带着弹簧般的弹跳落定
