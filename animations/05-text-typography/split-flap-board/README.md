# 翻牌器

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/split-flap-board/)

## 它是什么
翻牌器让每个字母显示在一块中间断开的翻牌上，就像老机场和火车站的出发牌。要换一个字母，翻牌就逐个字母翻过字母表：旧字母的上半部分向前翻过铰链，下一个字母的下半部分落在它下面，一次又一次，直到翻牌显示新字母。字母在字母表中越靠后，翻牌翻得越久，所以词是一块翻牌一块翻牌地定下来的。

## 什么时候用
- 旅行、交通或复古主题的主视觉标题和开场
- 登机口、状态和到达牌，或者一条时不时变化的“正在放映”文字
- 会变的短词：一个目的地、一个状态、一个分数（在字母序列里加上数字）
- 应该感觉机械、又有点俏皮的时刻

## 工作原理
每块翻牌是四个同样大小和颜色的层。两个静止的半块填满翻牌；两个翻页盖在上面，绕着中间的铰链转动。每个半块是一个翻牌一半高的盒子，溢出被藏起来，里面放着一个和整块翻牌一样高的字母盒，固定在顶部或底部，所以上半块显示字母的上半部分，下半块显示字母的下半部分：

```css
.tile { position: relative; width: var(--tw); height: calc(var(--tw) * 1.4); perspective: calc(var(--tw) * 5); line-height: 1; }
.half, .flap { position: absolute; left: 0; right: 0; height: 50%; overflow: hidden; background-color: var(--tile); }
.up { top: 0; }  .down { bottom: 0; }
.half b, .flap b { position: absolute; left: 0; right: 0; height: 200%; display: flex; align-items: center; justify-content: center; }
.up b { top: 0; }  .down b { bottom: 0; }
.flap.up { transform-origin: 50% 100%; }    /* turns on its bottom edge: the hinge */
.flap.down { transform-origin: 50% 0; }     /* turns on its top edge: the hinge */
```

从一个字母翻到下一个字母时，静止的上半块已经显示下一个字母，静止的下半块仍然显示旧字母。显示旧字母上半部分的上面翻页从竖直加速翻到边对边；然后显示下一个字母下半部分的下面翻页从边对边减速摆进原位。每个翻页上的暗层转得越远就越强，就像它转离了光线。字母只在翻牌开始新一翻时才写；在那之间，只有两个 `rotateX()` 变换在变。

一个帧循环驱动所有翻牌。时钟按上一帧以来的时间推进（最多 50ms，慢动作下是三分之一的速度），每块翻牌由此算出自己在第几翻、翻到多远，所以在 60 Hz 和 120 Hz 的屏幕上，翻牌的速度是一样的：

```js
const local = clock - i * STAGGER;                 // each tile starts a moment after the one on its left
const k = Math.floor(local / flipMs);              // the flip it is on
if (k >= flips) { rest(t, t.seq[flips]); return; } // done: its own letter
const p = local / flipMs - k;                      // how far through that flip, 0 to 1
if (p < .5) { const q = p * 2; turn(t.f1, t.s1, -90 * q * q); }          // the old letter's top falls
else { const q = (p - .5) * 2; turn(t.f2, t.s2, 90 * (1 - q) * (1 - q)); } // the next letter's bottom lands
```

每块翻牌的字母序列是从空白到它字母的转盘（“ ABC…”一直到它），所以显示 A 的翻牌翻一次，显示 Z 的翻牌翻 26 次。减弱动态效果下什么都不转：字母一块一块淡入。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 词语 | BOARDING | 牌上的词：最多 10 个 A 到 Z 的字母，空格会留出一块空白翻牌 |
| 翻牌速度 | 普通 | 一翻需要的时间：慢速 150ms，普通 85ms，快速 45ms；每块翻牌比它左边的晚 40ms 开始 |
| 翻牌颜色 | 黑色 | 翻牌的颜色：黑色、蓝色、红色或绿色配浅色字母，或者奶油色配深色字母 |

## 生产注意事项
- **转盘**：真正的牌只向前翻，所以从 Z 回到 A 要经过每个字母。给时间和登机口、分数用的字母序列加上数字和符号；顺序决定每次变化需要的时间。从一个词换到另一个词时，让每块翻牌从它现在显示的字母开始，而不是从空白开始。
- **只用变换**：翻页用 `rotateX()` 转动，阴影是一个透明度，所以翻牌转动时什么都不用重新布局。字母每翻写一次，绝不是每帧都写。
- **精确的对半**：字母的两半必须在铰链处接合。用 `line-height: 1`、一个正好是半块两倍高的字母盒，并检查字体的大写字母：有些坐得稍高或稍低，需要小小的调整。
- **无障碍**：整行是一个用最终词命名的图片，翻牌对屏幕阅读器隐藏，所以词只读一次，而不是读它经过的每个字母。
- **声音**：柔和的咔哒声让牌感觉真实，但只在访客互动之后播放，并给他们一个静音的办法。
- **库**：GSAP 可以按翻牌排列 `rotationX` 补间；也有小的翻牌 web 组件。纯 CSS 版本也行：给每块翻牌一列字母，用 `steps()` 移动它，就像手册首页上的卡片那样，这样每块翻牌停在自己的字母上。要注意的是，关键帧和步数必须为每个词生成。

## 另见
- [乱码/故障文字](../scramble-text/) — 随机符号从左到右逐个锁定为真实文字
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
- [翻转进入](../../02-entrance-and-exit/flip-in/) — 像卡片翻过来一样用 3D 摆进视野
- [3D 翻转卡片](../../06-3d-advanced/flip-card-3d/) — 一张卡片在 3D 里翻过来，露出背面
- [数字滚动](../../04-micro-interactions/rolling-numbers/) — 数字滚到新值，像里程表
