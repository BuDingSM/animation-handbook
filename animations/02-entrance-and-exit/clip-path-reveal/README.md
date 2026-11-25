# 剪裁路径揭示

[在线演示](https://animation.xizanghuapi.cn/animations/02-entrance-and-exit/clip-path-reveal/)

## 它是什么

剪裁路径揭示通过让一个遮罩形状不断变大来揭开元素，直到整个元素都显示出来。元素全程都是完整绘制的，只有可见部分在变大。因为什么都不移动、不淡入淡出，内容在形状擦过去时保持清晰、不动，这让这个效果有了电影感和编辑感。

## 什么时候用
- 滚动时的图片和媒体揭示（照片从左到右被擦开）
- 主视觉区块，方向性的擦除揭开标题块
- 从点击处扩大的圆形或「虹膜」揭示
- 通栏面板之间的切换，用形状遮罩代替交叉淡入淡出

## 工作原理
表面带有一个起始 `clip-path` 把它藏起来，加上 `.in` 时过渡到打开的那个。两个值都存在 CSS 变量里，形状和方向控件可以替换它们：

```css
.surface {
  clip-path: var(--cp-out, inset(0 100% 0 0));
  transition: clip-path var(--dur) var(--ease);
  will-change: clip-path;
}
.surface.in { clip-path: var(--cp-in, inset(0 0% 0 0)); }
```

演示里有一张形状/方向配对的查找表——`inset()` 的值从一条边擦除，而 `circle()` 和 `ellipse()` 从中心向外扩大半径：

```js
const CLIPS = {
  inset: {
    left:  { out: 'inset(0 100% 0 0)', in: 'inset(0 0% 0 0)' },
    right: { out: 'inset(0 0 0 100%)', in: 'inset(0 0 0 0%)' },
    center:{ out: 'inset(50% 50%)',    in: 'inset(0%)' },
  },
  circle:  { out: 'circle(0% at 50% 50%)',      in: 'circle(75% at 50% 50%)' },
  ellipse: { out: 'ellipse(0% 0% at 50% 50%)',  in: 'ellipse(80% 70% at 50% 50%)' },
};
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 形状 | 边缘擦除 | 「边缘擦除」从一边或从中间揭开它；「圆形」和「椭圆形」像虹膜一样从中间打开 |
| 从哪开始 | 左 | 对边缘擦除来说，揭示从哪里开始：左、右、上、下或中间 |
| 速度 | 普通 | 揭示要多长时间：「慢」是 1300ms，「普通」800ms，「快」500ms；揭示比淡入稍慢一点，读起来最好 |
| 感觉 | 平滑 | 「平滑」慢慢停下，而不是戛然而止；「均匀」保持一个稳定的速度 |

## 生产注意事项
- **只在兼容的形状之间插值。**CSS 可以给 `inset`→`inset`、`circle`→`circle` 或 `polygon`→`polygon`（顶点数相同）做动画，但不能在形状函数*之间*补间。像演示这样，让 `out` 和 `in` 保持同一个函数。
- **`clip-path` 是绘制，不是合成。**它比给布局做动画便宜得多，但比 `opacity`/`transform` 重；在一个巨大的全屏元素上用复杂的 `polygon` 时，在手机上测试一下，掉帧就简化形状。
- **被裁剪的内容留在无障碍树里。**和幕布覆盖层不同，clip-path 下面的文字全程都还在、还能选中——对 SEO 和屏幕阅读器是好事，但也意味着「藏着」的内容对辅助技术来说并不是真的藏着。
- **减弱动态：**在 `prefers-reduced-motion` 下，演示去掉裁剪动画，退回到不透明度淡入淡出，不播放擦除。
- **库里的等价写法：**GSAP 的 `gsap.to(el, { clipPath: 'inset(0 0% 0 0)' })`；Framer Motion 给 `clipPath` 字符串做动画；Motion One 的 `animate(el, { clipPath: [...] })`。GSAP 老的做法给 Safari 用 `-webkit-clip-path`——现代目标不再需要这个前缀。

## 另见
- [幕布揭示](../curtain-reveal/) — 一块彩色面板盖住它，然后滑走
- [上滑揭示](../slide-up-reveal/) — 文字从一条看不见的边缘后方升起
- [拆字揭示](../split-text-reveal/) — 文字一块一块出现
- [淡入淡出](../fade-in-out/) — 单纯的淡入淡出，而不是形状
- [墨水扩散揭示](../ink-spread-reveal/) — 墨渍扩散、汇合，揭开一张图片
