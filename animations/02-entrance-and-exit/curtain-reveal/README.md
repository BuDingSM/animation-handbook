# 幕布揭示

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/curtain-reveal/)

## 它是什么

幕布揭示分三步：一块纯色面板滑过内容盖住它，停顿，然后从另一边滑走，露出它。和剪裁路径揭示不同，内容是真的藏在移动的面板后面再被揭开，所以眼睛会注意到一个清晰的前后变化。它是故意做得戏剧化的，适合配得上这种戏剧感的内容。

## 什么时候用
- 页面和章节切换，彩色面板在视图之间擦过
- 主视觉开场，把标题或 Logo 的第一次出现戏剧化
- 「分开」的幕布效果，两半分开，露出后面的东西
- 作品集和编辑类网站，过渡本身就是品牌的一部分

## 工作原理
幕布是一条绝对定位的横条，在 `z-index: 2` 上盖在内容上方，完全用 `transform: translateX/Y` 移动。它从屏幕外开始，滑到 `translateX(0)` 盖住，再滑到 `translateX(101%)` 离开。三个位置由类来驱动：

```css
.curtain[data-axis=h] { inset: 0 auto 0 0; width: 100%; transform: translateX(-101%); }
.curtain[data-axis=h].c-in  { transform: translateX(0); }   /* covers content */
.curtain[data-axis=h].c-out { transform: translateX(101%); } /* exits far side */
```

JavaScript 用嵌套计时器安排各个阶段——进入、按 `pause` 停住、再退出——所以揭示发生在*横条还在屏幕上*的时候：

```js
curtain.classList.add('c-in');                 // stage A: cover
setTimeout(() => {                             // stage B: pause
  setTimeout(() => {                           // stage C: uncover
    curtain.classList.remove('c-in');
    curtain.classList.add('c-out');
  }, pause);
}, dur);
```

「两边」用两块半宽的面板，从相对的边缘出发，在中间汇合再回去。`101%`（而不是 `100%`）保证横条完全越过边缘，不留亚像素的缝。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 从哪边进入 | 左 | 面板往哪个方向走；「两边」会派出两块在中间汇合再分开的面板 |
| 速度 | 普通 | 一趟的时间：「慢」是 800ms，「普通」500ms，「快」300ms；整个揭示大约是两趟加上停顿 |
| 遮住时停顿 | 短 | 面板盖住一切的时间：无、短（300ms）或长（800ms）；停顿就是让变化被注意到的关键 |
| 幕布颜色 | 深色 | 面板的颜色；与内容对比强烈会让揭示更夸张 |

## 生产注意事项
- **用 `101%` 超过边缘。**退出正好停在 `100%` 时，在分数 DPI 的显示器上可能留下一条 1px 的缝；多出来的这百分之一是标准的修法。
- **后面的内容永远不会从 DOM 里藏起来**——只是视觉上被盖住。如果揭示是给*加载*把门的，在 `pause` 阶段、横条遮住切换的时候换掉下面的内容，这样用户永远看不到变化发生的那一刻。
- **防止重入。**每次播放都会清除上一次的计时器，这样第二次触发就不能在动画中途开始一个新序列、打乱各个阶段——对任何多阶段、计时器驱动的效果都很重要。
- **减弱动态：**在 `prefers-reduced-motion` 下，播放会完全跳过盖住再揭开的过程——内容一直保持可见，面板永远不会从它上面闪过。
- **库里的等价写法：**GSAP 的时间线最合适——`.to(curtain, { x: 0 }).to(curtain, { x: '101%' }, '+=0.3')` 用内置的停顿把各个阶段串起来；Framer Motion 通过 `AnimatePresence` 给变体排顺序；Motion One 的 `animate` 接受带 `offset` 和 `delay` 的关键帧数组，来安排这三拍。

## 另见
- [剪裁路径揭示](../clip-path-reveal/) — 一个形状揭开它，没有面板
- [上滑揭示](../slide-up-reveal/) — 文字从一条看不见的边缘后方升起
- [滑入](../slide-in/) — 从一边滑入到位
- [拆字揭示](../split-text-reveal/) — 文字一块一块出现
- [覆盖擦除](../../03-page-transitions/overlay-wipe/) — 一块彩色面板遮住整个页面切换
