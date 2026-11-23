# 滚动图片序列

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-image-sequence/)

## 它是什么
滚动图片序列在你滚动时播放一串静止图片，就像翻一本翻页书：每个滚动位置显示一张图片，所以往下滚播放动画，往上滚倒放。产品页面用它把一段预渲染的动画（比如一部旋转的手机）变成读者能控制的东西。演示边走边画图片——一个旋转的线框立方体，所以不需要图片文件。

## 什么时候用
- 用户滚动时，实物旋转或拆解的产品展示
- 把一段短的预渲染 3D 动画变成可交互、可用滚动 scrub 的主视觉
- 滚动速度等于播放速度的新手引导或讲解区域
- 任何想要逐帧精确控制、而不是 CSS 过渡的“电影感”滚动时刻

## 工作原理
一个很高的轨道元素提供滚动预算；一个子元素用 `position: sticky` 把自己固定住。每次滚动事件都把轨道的位置换算成 `0 → 1` 的进度值，再乘以帧数；一个帧循环画出那一帧。这个演示没有图片素材，每帧都是程序生成的——但 scrub 逻辑与生产版本完全相同。

```js
function updateTarget(){
  const span = track.offsetHeight - stage.clientHeight;      // the stage is the scroll box
  const p = span > 0 ? Math.min(Math.max((stage.scrollTop - track.offsetTop) / span, 0), 1) : 0;
  targetFrame = p * (TOTAL - 1);
}
stage.addEventListener('scroll', updateTarget, {passive:true}); // fires for wheel AND touch
```

渲染循环要么直接跳到目标帧，要么缓缓滑向它（“在帧之间滑动”设置）：

```js
const dt = last ? Math.min(now - last, 50) : FRAME;  // FRAME = 1000 / 60; time since the last frame
last = now;
if (smoothing) shownFrame += (targetFrame - shownFrame) * (1 - Math.pow(1 - EASE, dt / FRAME)); // lerp; EASE 0.18
else           shownFrame  = targetFrame;                        // snap
const index = Math.round(shownFrame);
if (index !== drawn) { drawFrame(index); drawn = index; }         // skip a picture that is already on the canvas
```

`EASE` 是 1/60 秒内走过的距离占比，持续了 `dt` 的一帧会走完剩下的 `1 - Math.pow(1 - EASE, dt / FRAME)`，所以在 30、60 或 120Hz 的屏幕上，图片追上滚动用的时间都一样（每帧固定占比的话，120Hz 只用一半时间，30Hz 要两倍时间）。循环永不停歇，所以 `last` 从一帧保留到下一帧；`dt` 上限 50ms，所以隐藏的标签页不会让图片跳一下。

`drawFrame(index)` 是帧号的纯函数——同样的帧号，同样的图片——这正是反向 scrub 能干净倒带的原因。这也是循环可以跳过画布上已有的图片的原因，让页面在手机上很省。回到顶部，以及从顶部重新开始的播放，都让盒子一步跳过去；点任意一个按钮都会立刻把显示的帧设为目标帧，所以图片不会倒放。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 帧数 | 普通 | 序列有多少张图片：少是 24 张，普通 120 张，多是 240 张；帧越多 scrub 越平滑，但生产中要加载的图片也越多 |
| 在帧之间滑动 | 开 | 图片缓缓滑向滚动指向的帧，而不是跳过去 |

## 生产注意事项
- **预加载每一帧。**真正的技术加载一个 `Image` 对象数组（视频导出成带编号的 JPG/WebP 序列）。永远不要按需取帧——解码停顿会造成明显的空档。在区域滚入视野之前就开始预加载，准备好之前显示一个加载器。
- **帧数与体积。**全宽的 120–300 帧是很多字节。用 WebP/AVIF，把最长边限制到显示尺寸，移动端考虑用更少的帧数。
- **画最近的帧，别给画布做动画。**每次滚动刻度你都在替换整个画布；这里面没有 CSS 过渡。平滑是靠缓和*帧号*实现的，而不是过渡像素。
- **`will-change` / 解码。**加载后调用 `img.decode()`，让每帧的第一次绘制不会卡顿。把画布按 `devicePixelRatio`（取上限）设为设备像素尺寸，避免在高 DPI 手机上过度绘制。
- **触摸滚动免费生效**，因为这个效果由 `scroll` 事件驱动，而不是 `wheel`。除非需要，否则避免用 `preventDefault` 劫持滚动的库。
- **等效的库：**GSAP **ScrollTrigger** 对帧号对象做 `scrub` 补间，是经典实现；搭配 **Lenis**（或 GSAP ScrollSmoother）做惯性平滑。`<canvas>` 加预加载的帧数组是不用依赖的同样思路。

## 另见
- [滚动 scrub 动画](../scrub-animation/) — 滚动位置让一架飞机沿路径移动
- [固定动画](../pin-animation/) — 这个效果依赖的固定
- [吸顶区域](../sticky-section/) — 整个区域固定不动，内容随之变化
- [图片缩放进入](../zoom-into-image/) — 图片随滚动打开
