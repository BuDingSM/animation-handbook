# 共享元素过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/shared-element-transition/)

## 它是什么
共享元素过渡让页面切换时有一块内容保持可见：网格里的缩略图会长大、移动，进入详情页顶部的大图，而不是两个页面简单地互换。感觉像是你跟着内容一起走，而不是跳到一个新屏幕。演示先量出图片的起点和终点，再把它的一个副本从一处移到另一处。

## 什么时候用
- 图库、商品网格和媒体库里的缩略图到详情页导航
- 卡片到弹窗的展开，卡片变成弹窗的页眉
- 同一个对象在「之前」和「之后」两个视图里都存在、你想保留身份认同的任何地方
- 移动应用风格的导航，空间连续性有助于定位

## 工作原理
FLIP 是 First、Last、Invert、Play 的缩写。记录缩略图的矩形（First），显示目的地并读取主图的矩形（Last），把一个固定副本放在页眉的矩形上，再用 transform 把它画回缩略图上（Invert），然后把那个 transform 过渡到无（Play）：

```js
function openDetail(i, dur, ease){                            // dur, ease: the chosen speed and feel
  const p = PROJECTS[i];                                      // the project that was clicked
  const thumbRect = thumbEls[i].getBoundingClientRect();      // FIRST
  detail.classList.add('active');
  const heroRect = detailHero.getBoundingClientRect();        // LAST
  // INVERT: the copy sits on the header's rectangle; a transform draws it back onto the thumbnail
  const sx = thumbRect.width / heroRect.width, sy = thumbRect.height / heroRect.height;
  flipEl.style.cssText =
    `background:${p.bg};top:${heroRect.top}px;left:${heroRect.left}px;`+
    `width:${heroRect.width}px;height:${heroRect.height}px;transform-origin:0 0;`+
    `transform:translate(${thumbRect.left-heroRect.left}px,${thumbRect.top-heroRect.top}px) scale(${sx},${sy});`+
    `border-radius:${10/sx}px/${10/sy}px;transition:none;display:block`;
  void flipEl.offsetWidth;                                     // commit the inverted start
  flipEl.style.transition =                                    // PLAY
    `transform ${dur}ms ${ease},border-radius ${dur}ms ${ease}`;
  flipEl.style.transform = 'none';
  flipEl.style.borderRadius = '0';
}
```

真实的网格在副本移动时淡出，详情正文稍晚一点淡入（`dur*0.6` 的延迟），这样主图先到了，文字再出现。放好副本后读一下 `offsetWidth`，会让浏览器在过渡开启前先应用倒置的起点——没有这一步，副本会直接跳到终点。圆角设为 `10/sx` 除以 `10/sy`，这样副本缩小时圆角看起来仍然是 10px。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 图片移动并长大需要多长时间：慢是 800ms，普通 500ms，快 300ms；300 到 600ms 感觉灵敏又不拖沓 |
| 感觉 | 平滑 | 平滑减速到停；弹性会长得比页眉的尺寸稍大一点，再回落到位 |

## 生产注意事项
- **先读后写**——把所有 `getBoundingClientRect()` 读操作批量放在改样式之前。读写交错会造成布局抖动，让动画卡顿。
- **副本是 `position:fixed`**，所以它在视口坐标里移动，不受下面页面的滚动和布局影响。它只用 `transform` 移动（一个位移加一个缩放），所以移动时浏览器不用重新布局；只有圆角会被重绘。缩放的副本会拉伸里面的内容，适合图片或纯色，不适合文字。
- **固定定位用视口坐标**，这就是演示直接读 `getBoundingClientRect()` 的原因。如果你的副本在一个经过变换或滚动的祖先元素里，偏移量必须调整。
- **减弱动态效果**会完全跳过副本，直接切入详情视图。
- **等价的库**：View Transitions API 通过给两个元素设置相同的 `view-transition-name` 原生实现这一点。Framer Motion 的 `layoutId` 和共享的 `<motion.*>` 元素能自动做 FLIP；GSAP 的 Flip 插件提供同样的量取-倒置-播放原语。

## 另见
- [View Transitions API](../view-transitions-api/) — 浏览器自己就能做到
- [FLIP 技巧](../flip-technique/) — 单独使用的量取再移动方法
- [传送门 / 隧道缩放](../portal-zoom/) — 下一页从点击的圆圈里打开
- [变形过渡](../morph-transition/) — 形状变化而不是移动
