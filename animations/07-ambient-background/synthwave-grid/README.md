# 合成波网格

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/synthwave-grid/)

## 它是什么
合成波网格是发光的霓虹地板，伸向地平线、稳稳地向观众滚动而来，在紫色天空下有一个条纹的、发光的太阳。它是 1980 年代专辑封面和街机游戏的复古样子，是 1980 年代对未来的想象，画成一个永远看不出重启动的循环。

## 什么时候用
- 复古或蒸汽波主题的音乐、游戏和活动落地页
- 想要强烈深度感和前进运动的主视觉背景
- 80 年代风格的产品发布、歌单或宣传画面
- 大展示文字后面的循环，网格在文字下方后退

## 工作原理
地板按透视画出：竖线从地平线上的消失点扇形展开，横线按幂曲线间隔，在地平线附近聚拢、在观众附近散开。滚动只是一个对线条数取模的偏移，这让循环无缝——一条到达底部的线和一条在地平线出现的新线是一样的：

```js
const hz = horizon(), cx = W / 2;
for (let i = 0; i < DENS; i++) {
  let p = ((i + offset) % DENS) / DENS;      // 0 at horizon -> 1 at viewer
  const y = hz + Math.pow(p, 2.2) * (H - hz); // perspective compression
  ctx.globalAlpha = Math.min(1, p * 1.6);     // fade in near the horizon
  ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
}
```

偏移随时间增长，不随帧数：每六十分之一秒加上 `speed * 0.02`（慢动作里是三分之一），不管需要多少帧，所以地板在 30 Hz 的手机和 144 Hz 的显示器上以相同速度滚动。正常速度是每秒 1.2 条线。开始、暂停或从隐藏标签页返回后的第一帧什么也不加，帧之间的长间隔最多算 50ms：

```js
const dt = last === null ? 0 : Math.min(now - last, 50);   // ms since the last frame
last = now;
offset += speed * 0.02 * dt / (1000 / 60) * (slow ? 1 / 3 : 1);
```

辉光是设为网格颜色的画布 `shadowBlur`；太阳是一个裁剪过的半圆，填垂直渐变，用背景色的间隙切出条纹。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 地板向观众滚动的速度：慢速是正常的 0.6 倍，快速是 1.6 倍；正常时每秒 1.2 条新线离开地平线，中等线条时每条约 13 秒到达底边 |
| 线条数量 | 中等 | 地板有多少条线：少是横向 10 条、朝地平线 15 条，中等 16 和 23 条，多是 24 和 35 条；更多线条让网格更细 |
| 网格颜色 | 粉色 | 发光线条的颜色：粉色、青色、紫色或橙色 |

## 生产注意事项
- **`shadowBlur` 很贵**：画布阴影辉光是较重的 2D 操作之一。这个线条数没问题，但如果大幅提高密度，就去掉阴影，用每条线第二次更粗、低透明度的描画假装辉光。
- **取模接缝**：`p` 接近 0 时把线条淡入，藏住新线在地平线生成处的突然出现。没有透明度渐变，你会看到它闪烁着出现。
- **减少动态**：演示开始时是暂停的，显示静止的网格、天空和太阳，直到访客按下播放。生产环境里，给这些访客显示静止场景。
- **CSS 替代方案**：这也可以用 `transform: perspective()` 平面和朝观众滚动的重复线性渐变来建。用 `background-position` 滚动它每帧都会重绘；用 `transform: translateY()` 把超大平面滑动一个网格、再跳回来，让合成器运行它。画布版本在逐线淡出和辉光的控制上胜出。
- **库的等价实现**：想要带相机移动和泛光的真正 3D 地板，用 [three.js](https://threejs.org)——带线框材质的 `PlaneGeometry` 加 `UnrealBloomPass` 给出真实的辉光。[tsParticles](https://github.com/tsparticles/tsparticles) 不适合结构化网格。

## 另见
- [星空](../starfield/) — 另一种向前飞越太空的方式
- [扫描线效果](../scanline/) — 完成老屏幕样子的深色线条
- [网格 / 圆点图案视差](../grid-dot-pattern-parallax/) — 相对指针移动的平面网格
- [动态渐变背景](../animated-gradient-background/) — 像天空一样的渐变，自己移动
