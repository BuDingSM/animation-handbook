# 手写描绘

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/handwriting-draw/)

## 它是什么
手写描绘让一个词看起来像有人在你眼前写出来。这个词由钢笔线条组成，而不是字体，每条线按人写字的顺序，从钢笔落笔的地方到抬起的地方揭开：先写连在一起的字母，再写 i 上的点和 t 上的横。

## 什么时候用
- 问候语、感谢信和邀请函，应该感觉亲切
- 信件、引用或“关于我”下面的签名
- 页面开场和带手工感的主视觉标题
- 手写风格的 Logo 和品牌标记

## 工作原理
每个词是一组 SVG 路径的列表，每笔一根，按书写顺序：对 "hi there" 来说，就是 hi、i 上的点、there，再是 t 上的横。字母是基线上的手绘三次曲线，已经倾斜了，所以不涉及字体文件。每条路径用 `getTotalLength()` 量出来，藏在一条和它一样长的虚线后面：

```js
const len = path.getTotalLength();
path.style.strokeDasharray = len + ' ' + len;
path.style.strokeDashoffset = len; // hidden: the gap covers the whole line
```

一个帧循环按过去的时间推动时钟（慢动作下是三分之一的速度），所以钢笔在任何屏幕上都保持速度。每笔按它长度除以运笔速度，占据时钟的一段，下一笔之前有短暂的抬笔，所以长笔画花的时间更长，和真正的钢笔一样：

```js
function frame(ts) {
  const dt = lastT === null ? 0 : Math.min(ts - lastT, 50);
  lastT = ts;
  t += dt * (slowMotion ? 1 / 3 : 1);
  strokes.forEach(s => {
    const p = Math.min(1, Math.max(0, (t - s.start) / s.dur));
    s.el.style.strokeDashoffset = s.len * (1 - ease(p));
  });
  if (t < total) requestAnimationFrame(frame);
}
```

`ease()` 让每笔开始和结束时都是平均速度的 40%。钢笔放在正在画的那笔的 `getPointAtLength()` 处；笔画之间，它从一笔的末端滑到下一笔的起点，略高于纸面。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 词语 | hello | 写哪个词：hello 是一笔，thanks 是两笔（词本身，再是 t 上的横），hi there 是四笔（hi、点、there、横） |
| 运笔速度 | 普通 | 钢笔每秒走多远：慢速是 200 单位，普通 400，快速 800，一个小字母是 28 单位高；一个词大约需要 1 到 5 秒 |
| 墨水颜色 | 蓝色 | 墨水的颜色，也是钢笔的颜色 |
| 笔尖粗细 | 中 | 线条有多宽：细是 1.1 单位，中 1.8，粗 2.8，一个小字母是 28 单位高 |
| 显示钢笔 | 开 | 一支小钢笔骑在线条的笔尖上，在笔画之间抬起来 |

## 生产注意事项
- **获取路径**：字体的字母是填充的轮廓，所以描它们的边缘会把每个字母描两遍，看起来完全不像写字。用单根开放的笔画代替：用钢笔工具画出来（Figma、Illustrator、Inkscape），用单线描摹扫描的签名，或者从单线字体开始，比如为绘图仪做的 Hershey 字体。保持每笔一根路径，按书写顺序，每根从钢笔落笔的地方开始。
- **按长度给时间**：给每笔一个和它长度成正比的时间，而不是一个固定时长，否则 i 上的点会和整个词花一样长的时间。一个最短时间（这里是 120 ms）让点保持可见。
- **减弱动态效果**：演示把写好的词淡入。生产中，把词完整地展示出来，或者淡入。
- **无障碍**：图画对屏幕阅读器来说是一张图片（`role="img"` 加词的 `aria-label`），所以它们只读一次这个词，而不是一串路径。
- **库的等价做法**：GSAP 的 DrawSVG 插件给虚线做动画（`drawSVG: "0%"` 到 `"100%"`），时间线把笔画排好顺序；Framer Motion 按路径给 `pathLength` 做动画；Vara.js 用它自己的单线字体写字；Lottie 播放从 After Effects 导出的手写。

## 另见
- [SVG 路径动画](../../06-3d-advanced/svg-path-animation/) — 一条线一笔一笔地画出自己
- [对勾绘制](../../04-micro-interactions/checkmark-draw/) — 同样的虚线技巧，用在一个成功对勾上
- [打字机效果](../typewriter-effect/) — 文字逐字打出
- [逐字母出现](../../02-entrance-and-exit/letter-by-letter-stagger/) — 短语逐字出现
