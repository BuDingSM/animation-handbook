# 灵动岛

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/dynamic-island/)

## 它是什么
灵动岛是手机屏幕顶部的一个小黑色药丸，会长成一张更大的卡片，显示实时内容，比如来电、计时器或正在播放的歌曲，完成后缩回药丸。药丸和卡片是一个黑色形状，轮廓会变化，所以变化读起来像一个物体在伸展，而不是打开了一个新窗口。名字来自 iPhone，但同样的想法适合应用或页面顶部任何状态区域。

## 什么时候用
- 网页或移动应用里的来电和视频通话
- 计时器、锻炼和倒计时，访客做别的事时它们还在走
- 带曲目和控制的音乐和播客播放器
- 几分钟内会变化的订单、配送和行程更新
- 实时比分和其他值得瞥一眼的短更新

## 工作原理
灵动岛是一个盒子，大小是它最大的卡片，再留一点空间给回弹。里面所有东西都按完整大小布局一次；访客看到的是一个用 `clip-path: inset()` 切出来的黑色层，每张卡片的内容都在这个层里面，所以同一个裁剪也切内容，生长或缩小时什么都不会显示在形状外面。三个自定义属性描述可见形状：一半宽度（`--hw`）、高度（`--h`）和圆角半径（`--r`）。改变它们就是整个动画：浏览器把裁剪从一个形状缓动到另一个形状，药丸长成卡片，或一张卡片直接变成另一张，永远不用动画 `width` 或 `height`。

```html
<button class="isl" type="button" aria-label="Phone notification" aria-expanded="false" data-n="call">
  <span class="shape">                                    <!-- the black layer, clipped -->
    <span class="c c-call" aria-hidden="true">…</span>  <!-- each card's content, at full size, clipped with it -->
  </span>
</button>
```

```css
.isl   { --hw: 48px; --h: 32px; --r: 16px; position: absolute; top: 11px; left: 6px; right: 6px; height: 172px; }
.shape { position: absolute; inset: 0; background: #000;
         clip-path: inset(0 calc(50% - var(--hw)) calc(100% - var(--h)) round var(--r));
         transition: clip-path 700ms var(--ease); }
/* The content fades in once the shape has room and scales evenly, so its text never stretches */
.c { opacity: 0; transform: scale(.88); transition: opacity 175ms ease, transform 700ms var(--ease); }
.open[data-n="call"] .c-call { opacity: 1; transform: none; transition-duration: 315ms, 700ms; transition-delay: 140ms; }
```

```js
// Each shape as [half its width, its height, its corner radius]; a % is of the island's box
const SIZE = { rest: ['48px', '32px', '16px'], call: ['44%', '76px', '30px'],
               timer: ['40%', '64px', '32px'], music: ['44%', '148px', '34px'] };
// note is the picked notification; data-n is the card on show. The shape takes the size of the card on show while it is open,
// else the resting pill's (under reduced motion the card keeps its size while it fades out)
function shape() {
  const g = SIZE[open || reduceMq.matches ? isl.dataset.n : 'rest'];
  ['--hw', '--h', '--r'].forEach((v, i) => isl.style.setProperty(v, g[i]));
}
function setOpen(next) {
  if (next) isl.dataset.n = note;   // under reduced motion only once the shape has faded out (see below)
  open = next;
  isl.classList.toggle('open', open);
  isl.setAttribute('aria-expanded', String(open));
  shape();
}

// Bounce: a damped spring (damping z: .68 for Soft, .5 for Springy) sampled into a CSS linear() curve
function spring(z) {
  const w = 4.6 / z, d = w * Math.sqrt(1 - z * z), p = [];
  for (let i = 0; i < 40; i++) {
    const t = i / 40;
    p.push((1 - Math.exp(-z * w * t) * (Math.cos(d * t) + z * w / d * Math.sin(d * t))).toFixed(3));
  }
  return 'linear(' + p.join(',') + ',1)';   // set as --ease on the island
}
```

同样的过渡在任意两个形状之间都一样，所以打开时换另一个通知，卡片会直接变形。裁剪也决定按钮哪里能按，所以小药丸得到一个更大的隐形目标（124 × 48px）。按钮就是整个盒子，它自己的轮廓跟不上形状：焦点环是第二个裁剪层，比形状大 3px，在它后面。强制颜色模式下手机保持自己的颜色，圆环也是如此：系统颜色比如高亮，在浅色主题里是深色，对着深色手机会消失，而圆环自己的橙色在每个主题里都对它显眼。轮廓保持透明，作为圆环保不住颜色时的回退。卡片打开时状态栏淡出；短屏幕上，锁屏日期在打开的音乐卡片下面，日期也会淡出（屏幕上的尺寸容器查询），所以卡片边缘永远不会切过它。礼貌的实时区域在访客打开时读出消息，而不是「演示一下」打开时。大屏幕上内容还会从轻微模糊中聚焦进来；手机跳过模糊。减弱动态效果下，静止的药丸留在原地，卡片以自己的大小在它上方淡入，所以什么都不生长、不移动。卡片只在形状完全淡出时才换：换另一个通知时，先把打开的卡片（或关闭后还在淡出的卡片）淡出，在隐藏时放好新内容和大小，再淡入。选中的通知会立刻保留，所以中途取消切换的按压，下次仍然打开选中的卡片；那里的焦点环是空心轮廓，所以透过淡出的卡片什么都看不见。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 通知类型 | 来电 | 96 × 32px 的药丸长成什么：76px 高的来电卡片、64px 高的宽胶囊计时器，或 148px 高、带控制的音乐卡片；来电和音乐卡片大约是屏幕宽度的 85%，计时器大约是 77%；打开时换另一个，形状会直接变成新的 |
| 速度 | 普通 | 长大或缩回要花多长时间：慢是 1100ms，普通 700ms，快 450ms；弹簧比普通缓动需要稍长一点，因为它在结尾落定 |
| 回弹 | 柔和 | 无：缓出，不超过大小；柔和：超过大小变化的约 5%，回弹：约 16%，各自在时间结束时落定 |

## 生产注意事项
- **绝不要动画 `width` 或 `height`**：每一帧都会重新布局卡片和文字，字会在生长时换行跳动。用裁剪，或者在图层上缩放、把子元素反向缩放回来，只改变画出来的东西。
- **裁剪会重绘**：`clip-path` 动画每一帧都会重绘灵动岛。这个大小的形状很便宜；大面板请在图层上动画 `transform: scale()`，并把内容反向缩放（FLIP 技巧），这样留在合成器上。
- **内容和形状一起裁剪**：把内容放在裁剪层里面，不要放在旁边。缩小的裁剪跑得比淡出快，所以盖在形状上的内容会在每次关闭开始时，在形状周围的页面上出现一会儿。
- **给回弹留空间**：弹簧会超过目标大小，裁剪显示不出盒子以外的东西。把盒子做得比最大的卡片大，超出回弹的量，否则角落会在峰值时有一瞬间变方。
- **`linear()` 支持**：弹簧曲线需要 `linear()`（Chrome 和 Edge 113、Firefox 112、Safari 17.2）。用 `CSS.supports()` 检查，回退到超过幅度差不多的 `cubic-bezier()`：`cubic-bezier(.3, 1.4, .5, 1)` 超过结尾约 5%，`cubic-bezier(.3, 1.75, .45, 1)` 约 16%。
- **点击区域和焦点**：裁剪也会裁剪指针命中。给小药丸至少 44 × 44px 的目标，焦点环画成自己的裁剪层。强制颜色模式下，如果灵动岛保持自己的颜色（`forced-color-adjust: none`），也让圆环保持自己的颜色：它的对比度是按你知道的颜色来衡量的，而系统颜色比如 `Highlight` 在浅色主题里是深色，对着深色灵动岛可能会消失。保留透明轮廓作为回退。
- **无障碍**：让药丸成为带 `aria-expanded` 的真正 `<button>`，在礼貌的 `role="status"` 区域里播报新消息，绝不要把焦点移进卡片，Esc 可以关掉它。
- **Framer Motion**：`<motion.div layout style={{ borderRadius: 30 }} />` 用变换动画尺寸变化，并保持圆角正确；子元素也加上 `layout`，这样它们会被反向缩放，文字不会拉伸。
- **GSAP**：Flip 插件记录药丸的盒子并动画到卡片；`gsap.to(shape, { clipPath: "inset(0% 6% 14% 6% round 30px)", ease: "elastic.out(1, 0.6)" })` 用回弹给裁剪本身做动画。
- **原生应用**：iOS 上真正的灵动岛通过 ActivityKit 显示实时活动；网页上它是一个你自己画的模式。

## 另见
- [轻提示](../toast-notification/) — 滑进角落、自己离开的短消息
- [搜索框展开](../expanding-search/) — 用裁剪打开成输入框的图标
- [按钮加载状态](../button-loading-states/) — 工作时被裁剪成圆形的按钮
- [弹窗展开](../modal-expand/) — 从按下的按钮处长出来的窗口
- [变形过渡](../../03-page-transitions/morph-transition/) — 随每页变化形状的标志
