# 渐变边框动画

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/gradient-border/)

## 它是什么
渐变边框动画是卡片或按钮周围的一条彩色边缘，一带颜色不停地绕着它流动。元素后面放着一个大图层，上面画着绕中心扫过的颜色；一个内部盒子盖住除了边缘以外的所有地方，这个图层转动，颜色就像沿着边框移动。

## 什么时候用
- 定价页面上的精选方案，或人们应该最先注意的那张卡片
- 主要的行动按钮，比如“升级”或“开始使用”
- 发布、新功能和限时优惠
- 在一小批卡片里标记选中的那张，要克制使用

## 工作原理
每个带边框的元素都包含一个圆环、一个辉光和一个内部盒子。圆环是一个和元素一样大的盒子，把里面的东西裁剪成元素的圆角形状；里面放着一个居中的正方形，宽度是元素的两倍，画着 `conic-gradient`。内部盒子按边框宽度向内缩进，涂着卡片自己的背景，所以只有正方形的边缘露出来。正方形用 `transform` 上的 CSS 动画转动：

```css
.gb      { position: relative; isolation: isolate; border-radius: 18px; }
.gb-ring { position: absolute; inset: 0; border-radius: inherit; overflow: hidden;
           background: rgba(255,255,255,.12); }          /* the faint line where the band is not */
.gb i    { position: absolute; left: 50%; top: 50%; width: 200%; aspect-ratio: 1;
           background: conic-gradient(var(--band));
           transform: translate(-50%, -50%);
           animation: gb-turn var(--dur) linear infinite; }
.gb-in   { position: relative; margin: var(--bw); border-radius: calc(18px - var(--bw));
           background: #121216; }                         /* covers all but the edge */
@keyframes gb-turn { to { transform: translate(-50%, -50%) rotate(1turn); } }

/* A long band fades in behind its bright front; All around loops back to the first color */
.stage         { --band: transparent 0 40%, var(--c1) 58%, var(--c2) 72%, var(--c3) 85%, var(--c4) 96%, transparent; }
.stage.b-full  { --band: var(--c1), var(--c2), var(--c3), var(--c4), var(--c1); }
```

正方形是元素宽度的两倍，这样它在任何角度都仍能盖住元素的角落（在元素高不到宽 1.7 倍时成立）。辉光是圆环的第二份复制品，放在元素后面（在元素自己的层叠上下文里 `z-index: -1`），带 `filter: blur(16px)`。关掉辉光时用 `visibility: hidden` 而不是 `display: none`，这样它的转动会和圆环保持同步。按钮的转动从圆圈的另一个点开始（`animation-delay: calc(var(--dur) * -.4)`），所以两条色带不会同步移动。

新的速度会先按比例缩放每圈已经播放的时间，再设置新的时长，这样色带会保持在原位，而不是跳动：

```js
stage.getAnimations({ subtree: true }).forEach(a => { a.currentTime = a.currentTime * next / secs; });
document.documentElement.style.setProperty('--dur', next + 's');
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 颜色 | 彩虹 | 色带的四种颜色：彩虹、日落（黄色到紫色）或海洋（青色到靛蓝） |
| 速度 | 普通 | 绕边缘一圈要花多长时间：慢是 7 秒，普通 4 秒，快是 2 秒；快得多会显得忙乱 |
| 柔光 | 开 | 元素后面一条色带的模糊复制品，让它的颜色柔和地溢到页面上 |
| 色带长度 | 长 | 颜色一次盖住边缘的多少：短色带大约绕三分之一圈，长色带大约绕五分之三，每条都从淡淡的尾巴渐显到明亮的前端；或者颜色整圈都有 |
| 边框宽度 | 中等 | 彩色边缘有多粗：细是 1px，中等 2px，粗是 4px |

## 生产注意事项
- **转动图层，绝不要转动渐变**：给渐变自己的角度做动画（用在 `conic-gradient(from var(--angle), …)` 里的 `@property --angle`）会在每一帧重绘元素。用 `transform` 转动一个画好的图层，工作就交给合成器。
- **按形状定图层大小**：宽度两倍于元素的正方形，只有在元素高不到宽 1.7 倍时，才能在任何角度都盖住它；更高的元素，按元素的对角线来定正方形的大小。
- **一屏一两个**：会动的边框很吸引眼睛。只给那一张精选卡片或主按钮用，不要给网格里的每张卡片都用。
- **手机上的模糊**：辉光的模糊是最耗性能的部分，所以演示在小屏幕上模糊得少一点，看起来同样柔和。
- **减弱动态效果**：在 `prefers-reduced-motion: reduce` 下，演示一开始是暂停的，所以边框的颜色是静止的。生产中把动画去掉，保留静止的渐变。
- **库**：GSAP 用 `gsap.to(layer, { rotation: 360, repeat: -1, ease: 'none', duration: 4 })` 转动图层；Framer Motion 用 `animate={{ rotate: 360 }}` 和 `transition={{ repeat: Infinity, ease: 'linear', duration: 4 }}` 做同样的事。Tailwind 的片段常常改用 `@property` 角度来做动画，能用，但每一帧都会重绘。

## 另见
- [微光扫过效果](../shimmer-effect/) — 一道光带横扫过去，而不是绕圈
- [通知徽标脉冲](../badge-pulse/) — 另一个吸引眼睛的小循环
- [文字渐变动画](../../05-text-typography/text-gradient-animation/) — 在字母里流动的颜色
- [动态渐变背景](../../07-ambient-background/animated-gradient-background/) — 整个缓慢变色的背景
- [聚光灯悬停辉光](../spotlight-hover/) — 指针靠近时卡片边缘亮起来
