# 翻转进入

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/flip-in/)

## 它是什么

翻转进入让元素从一个转开的角度以 3D 方式摆进完整视野，就像一块带铰链的面板转向你。关键细节是 3D 透视设在元素的容器上，而不是卡片本身；没有它，转动会塌成扁平的挤压，没有深度感。

## 什么时候用
- 仪表盘、画廊和新手引导流程里的卡片入场
- 揭示概念上有两面的东西的「正面」（闪卡、瓷贴）
- 模态框或面板，应该感觉是实实在在地到达，而不是淡入
- 任何想用一点点立体感把一个元素和扁平布局区分开的地方

## 工作原理
舞台设置 `perspective`，给它的子元素建立一个共享的消失点。卡片从旋转（默认 90°）开始，加上 `.in` 时过渡到 0°。因为透视是从父元素的 3D 上下文继承来的，旋转渲染成一次真正的摆动：

```css
.stage{perspective:var(--persp)}
.card{
  opacity:0;
  transform:rotateY(var(--rot-y)) rotateX(var(--rot-x));
  transform-origin:var(--origin);
  transition:transform var(--dur) var(--ease),opacity var(--dur) var(--ease)}
.card.in{opacity:1;transform:rotateY(0deg) rotateX(0deg)}
```

「摆动方向」设置决定哪个旋转带起始角度：「左右」用 rotateY（竖直的铰链，像一扇门），「上下」用 rotateX（平放的铰链，像一块翻盖），「对角」两个都用：

```js
const ry=axis==='X'?'0deg':rot+'deg';
const rx=axis==='Y'?'0deg':rot+'deg';
document.documentElement.style.setProperty('--rot-y',ry);
document.documentElement.style.setProperty('--rot-x',rx);
```

「铰链」设置把 transform-origin 移到一条边上（上、下、左），卡片就从那一边摆过来，而不是绕着中心转。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 摆动方向 | 左右 | 往哪个方向摆：像门一样绕竖直铰链左右摆，像翻盖一样上下摆，或者对角摆 |
| 铰链 | 中间 | 从哪里转；铰链在边上会让它从那一边摆过来 |
| 速度 | 普通 | 摆动要多长时间：「慢」是 1000ms，「普通」600ms，「快」350ms |
| 转多大 | 四分之一圈 | 开始的角度：「一小点」（45°）、「四分之一圈」（90°，侧面对着你）或「半圈」（180°，背对着你） |
| 3D 深度 | 中等 | 观众看起来有多近：「微妙」是 2000px，「中等」1000px，「强」400px；越近越夸张 |
| 感觉 | 平滑 | 「平滑」在摆到正对你时放慢；「弹回」加一点落定；「轻柔」两头放慢；「均匀」保持一个稳定的速度 |
| 淡入 | 开 | 转动时让它淡入 |

## 生产注意事项
- **透视属于父元素**：这是最常见的错误。在卡片本身上设置 `perspective`（通过 `transform: perspective(...)`）是按元素生效的，不会在兄弟元素之间共享消失点；容器的 `perspective` 属性才是创建连贯 3D 场景的东西。
- **大角度时的背面**：超过大约 90°，你会看到元素的背面。如果反面不应该显示，加 `backface-visibility:hidden`，或者给真正的双面翻转加第二个面。
- **`will-change:transform`**：把卡片提升到自己的图层，让 3D transform 保持平滑；演示提前就设置了它。
- **减弱动态**：在 `prefers-reduced-motion` 下，演示把翻转换成单纯的 300ms 不透明度淡入淡出，没有 transform——3D 旋转是很强的前庭触发因素。
- **Framer Motion**：在 `motion.div` 上给 `rotateY` 做动画，设置 `style={{ transformPerspective: 1000 }}` 或者在父元素上设置 `perspective`。
- **GSAP**：设置 `transformPerspective`（或者容器上的 `perspective`），给 `rotationY` 做补间；GSAP 内置了 3D 处理。

## 另见
- [缩放进入](../scale-in/) — 放大到位，在页面上是平的
- [旋转进入](../rotate-in/) — 平着旋转，而不是 3D 转动
- [模糊渐入](../blur-in/) — 淡入时从模糊中变清晰
