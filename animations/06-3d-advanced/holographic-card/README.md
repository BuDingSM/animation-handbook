# 全息卡片

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/holographic-card/)

## 它是什么
全息卡片模仿稀有收藏卡上的彩虹箔片。卡片朝指针倾斜时，一片彩虹色滑过卡面，照亮下面的图片，就像真正的箔片迎着光，一处光斑跟着指针走。它是一叠平面图层，靠浏览器自己的 3D 变换移动，没有图片文件，也没有 3D 库。

## 什么时候用
- 集换式卡牌游戏和收藏品页面，箔片标志着稀有卡
- 应该显得高端的会员卡、忠诚卡和礼品卡
- 用户在个人资料里炫耀的徽章和奖励
- 邀请人凑近细看的产品发布、门票或通行证卡片

## 工作原理
卡片是叠在一个盒子里的四层，每层都裁剪到卡片的圆角：图片、箔片、光斑和文字面板。演示的图片画成内联 SVG；真实网站上它是一张图片。

```html
<div class="card">
  <div class="ly art">…the picture…</div>
  <div class="ly foil"><div class="sheen"></div></div>
  <div class="ly gl"><div class="spot"></div></div>
  <div class="ly txt">…the text panels…
    <div class="spot gloss"></div>   <!-- a faint copy of the glare, over the panels -->
  </div>
</div>
```

箔片是一片带透明间隙的彩虹条纹，有卡片两倍大，留出滑动的空间。`mix-blend-mode: color-dodge` 让它提亮每条条纹下面的东西，所以图片暗的地方只闪一点，亮的地方会爆出颜色，这正是它读起来像箔片而不是彩色薄膜的原因。光斑是一处和卡片等大的白色光点，中间最亮；`mix-blend-mode: plus-lighter` 把它的光加到图片上，所以它读起来是指针下的高光，而不是雾气。文字面板接着来，是实心的，所以箔片和完整的光斑永远不会压在文字下面。光斑的一个淡淡副本——光泽——以最多光斑四分之一的强度盖在面板上，同样照亮它们，让最暗的文字保持在 5:1 以上：

```css
.stage { perspective: 1000px;                       /* depth for the tilt, on the parent */
  --hc-c1: #f0648c; --hc-c2: #f4c35e; --hc-c3: #71e2a2; --hc-c4: #5ac2f4; --hc-c5: #a888f6;   /* the colors, in one place */
  --hc-light: 255,255,255; --hc-panel: #110e27; }
.card  { position: relative; aspect-ratio: 5 / 7; border-radius: 12px; will-change: transform; }
.ly    { position: absolute; inset: 0; border-radius: inherit; overflow: hidden; }
.foil  { mix-blend-mode: color-dodge; }
.sheen { position: absolute; inset: -50%; will-change: transform;
  background: repeating-linear-gradient(118deg, transparent 0%, var(--hc-c1) 3%, var(--hc-c2) 6%, var(--hc-c3) 9%,
    var(--hc-c4) 12%, var(--hc-c5) 15%, transparent 18%, transparent 21%); }
.gl    { mix-blend-mode: plus-lighter; }
.spot  { position: absolute; inset: 0; will-change: transform; opacity: 0;   /* the glare, and the gloss in the text layer */
  background: radial-gradient(circle closest-side, rgb(var(--hc-light)), rgba(var(--hc-light),.55) 22%,
    rgba(var(--hc-light),.18) 58%, transparent); }
.np, .ip { background: var(--hc-panel); }           /* the solid text panels */
```

每次指针移动变成三个数字：`x` 和 `y`，指针在卡片上的位置，从 -0.5 到 0.5；`h`，指针在卡片上时是 1，离开时是 0。一个函数画出它们，只改变换和不透明度，所以浏览器移动的是已经画好的图层，而不是重新绘制：

```js
// T: the lean at the card's edges in degrees (How far it tilts). G: the glare's strength (Glare).
// f0, f1: the foil's strength at rest and what lighting it adds.
card.style.transform  = `rotateX(${-2 * T * y}deg) rotateY(${2 * T * x}deg) scale(${1 + 0.04 * h})`;
sheen.style.transform = `translate(${-40 * x}%, ${-40 * y}%)`;   // the foil slides against the tilt
foil.style.opacity    = f0 + f1 * h;                              // and brightens while the card is lit
spot.style.transform  = gloss.style.transform = `translate(${100 * x}%, ${100 * y}%)`;  // the glare's middle under the pointer
spot.style.opacity    = G * h;
gloss.style.opacity   = 0.25 * G * h;                             // a quarter at most, over the text panels
```

显示的数字会滑向指针的数字，而不是跳过去。每帧按流逝的每六十分之一秒弥合 14% 的差距，所以滑行在 60、120 或 30 Hz 的屏幕上耗时相同，数字一到就停下循环：

```js
const k = 1 - Math.pow(1 - 0.14, dt / (1000 / 60));   // dt: the milliseconds since the last frame
sx += (ax - sx) * k;  sy += (ay - sy) * k;  sh += (ah - sh) * k;
```

指针按卡片自己的位置（未变换的盒子）测量，而不是按用户看到的倾斜轮廓，后者会在静止的指针下面移动，让卡片在边缘处抽搐。箔片图案是箔片层上的遮罩：同一片滑动的彩虹，只透过散落的圆点（六种不同大小的圆点平铺，看不出网格）或细斜线看到。遮罩不动，彩片在下面滑动，所以每个圆点或线条都会在条纹经过下面时闪亮：

```css
.card[data-foil=glitter] .foil { mask: radial-gradient(circle at 30% 40%, #000 0 1px, transparent 1.7px) 0 0 / 17px 23px,
                                       radial-gradient(circle at 75% 15%, #000 0 1px, transparent 1.7px) 0 0 / 23px 19px /* … */; }
.card[data-foil=lines]   .foil { mask: repeating-linear-gradient(60deg, #000 0 1.4px, transparent 1.4px 4.2px); }
```

**「显示图层」**把卡片转向一侧，给它 `transform-style: preserve-3d`，按每层在叠放中的位置把每层向前带 30px：当数字 `sp` 从 0 滑到 1 时，脚本在第 `i` 层上写 `translateZ(${30 * i * sp}px)`。只在图层分开或合拢时才写这些；卡片上的自定义属性也可以承载 `sp`，但卡片每帧都有新样式，每次都会传给每一层。箔片和光斑在那里不带混合模式绘制，所以每片都按原样显示；光斑在静止时保持亮着，偏上偏右，盖过图片的地方，而光泽被省掉。每层自己裁剪，这正是卡片能在 3D 里装下它们的原因：卡片本身的 `overflow: hidden` 会把它们压回一个平面。

**「演示一下」**让同样的数字沿 8 字形走 3.2 秒（`x = 0.42 sin 2πe`，`y = 0.34 sin 4πe`，`e` 是缓动后的进度），一路亮着，最后卡片放平。舞台里的一按、一键或滚轮转动，或用户自己的指针移过卡面，都会停掉这次运行，卡片从停下的地方开始跟随用户。

卡片也可以获得键盘焦点，方向键代替指针。键盘焦点在瞄准的地方点亮卡片（静止时是中间），每按一次方向键，瞄准点就向该方向移动卡片宽度的 6%（按住会重复移动），按 Escape 或把焦点移走会让它落定，和指针离开时一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 箔片图案 | 平滑 | 平滑把彩虹显示为柔和的色带；闪粉让它透过散落的圆点，细线让它透过细斜线，像真实卡片上的闪粉箔和蚀刻箔 |
| 倾斜幅度 | 中等 | 卡片在边缘处倾斜多远：轻柔是 6°，中等 12°，强烈 20°；再大一些，卡片上的文字在倾斜中途就难读了 |
| 光斑 | 柔和 | 指针下的光斑：关、柔和（55% 强度，略过一半）或明亮（全强度）；以四分之一强度的副本照亮文字面板 |
| 显示图层 | 关 | 把卡片转向一侧，把四层分开，图片、箔片、光斑和文字面板一层层叠着显示 |

## 生产注意事项
- **性能**：只有变换和不透明度在变，所以跟随指针不会让卡片逐帧重绘。许多全息卡片演示用 CSS 变量驱动 `background-position` 来移动渐变，每次指针移动都会重绘整张卡片、每一层：手机上看得出来，在网格里还会累加。移动图层上的混合模式仍然每帧耗一点显卡，所以只给指针下的卡片做动画，网格里其余的保持平放。
- **混合留在卡片上**：`mix-blend-mode` 把一层与它所在层叠上下文里下面画好的所有东西混合。卡片的变换让它自己成为层叠上下文，所以箔片只与卡片的图片混合；静止时没有变换的卡片需要 `isolation: isolate`，否则箔片也会照亮卡片后面的页面。
- **文字对比度**：文字放在箔片和光斑之上的实心面板上。颜色减淡会在亮条纹下面把深色背景提亮很多，加上光斑会把它变白，所以直接放在箔片上的文字在某些角度会失去对比度。面板本身的光必须保持微弱：演示最暗的文字（#110e27 上的 #bdb7e6）上四分之一强度的白光，仍然留有 5.4:1。
- **触摸**：Pointer Events 覆盖鼠标、手写笔和触摸。卡片上的 `touch-action: none` 让手指拖动时倾斜卡片而不是滚动页面；减弱动态下演示把它设回 `auto`，因为卡片不再倾斜。滑出卡片的手指会让卡片一直朝那条边倾斜，直到抬起。
- **遮罩和混合**：120 之前的 Chrome 和 Edge、15.4 之前的 Safari 需要 `-webkit-mask` 前缀；演示两种都写了。`plus-lighter` 是这些混合模式里最新的（Chrome 和 Firefox 在 2022 年加入）；没有它的浏览器用普通混合画光斑，淡一些但位置相同。
- **库**：VanillaTilt.js 做倾斜和光斑（`glare: true`、`"max-glare"`），Atropos 在不同深度加图层；两者都把箔片留给你的 CSS。React 里，Framer Motion 的 `useMotionValue` 和 `useSpring` 驱动同样的变换，GSAP 的 `quickTo` 给出同样的缓动跟随。
- **减弱动态**：卡片保持平放，什么也不滑动。指向它或按下「演示一下」，箔片和光斑会在原地变亮，然后再暗回去。

## 另见
- [视差 3D 倾斜](../parallax-3d-tilt/) — 同样朝指针倾斜，用普通的光而不是箔片
- [3D 翻转卡片](../flip-card-3d/) — 一张卡片在 3D 中翻过来露出背面
- [聚光灯悬停辉光](../../04-micro-interactions/spotlight-hover/) — 柔和的光晕跟随指针划过卡片网格
- [渐变边框动画](../../04-micro-interactions/gradient-border/) — 一条彩色带绕着卡片边缘流动
