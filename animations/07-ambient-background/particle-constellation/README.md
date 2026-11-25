# 粒子星座

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/particle-constellation/)

## 它是什么
粒子星座是一片小圆点在背景上慢慢漂移，任何两个靠得够近的圆点都由一条细线连起来，圆点分开时线条淡出。线条不断形成、断开，所以网络似乎在移动、生长：许多科技和产品网站后面的“相连圆点”背景。

## 什么时候用
- 科技、网络、安全或数据产品的主视觉区
- 需要生气、又不要焦点的加载和闲置状态
- 标题文字后面的背景，微妙的运动增加深度
- “相连系统”的隐喻能强化信息的任何地方

## 工作原理
节点是带位置和速度的普通对象；它们漂移、碰到边缘反弹。每帧，双重循环测试每一对——平方距离低于连线阈值时，画一条不透明度与接近程度成正比的线。用平方距离避免了拒绝情况下的 `sqrt`：

```js
const L2 = LINK * LINK;
ctx.strokeStyle = `rgb(${c[0]},${c[1]},${c[2]})`;
for (let i = 0; i < nodes.length; i++) {
  const a = nodes[i];
  for (let j = i + 1; j < nodes.length; j++) {
    const b = nodes[j], dx = a.x - b.x, dy = a.y - b.y, d2 = dx*dx + dy*dy;
    if (d2 < L2) {
      ctx.globalAlpha = (1 - Math.sqrt(d2) / LINK) * 0.55;
      ctx.beginPath(); ctx.moveTo(a.x, a.y); ctx.lineTo(b.x, b.y); ctx.stroke();
    }
  }
}
ctx.globalAlpha = 1;
```

每条线共享一种描画颜色，用 `globalAlpha` 实现淡出；为每条线构建新的颜色字符串要贵几倍。

可选的鼠标吸引把每个节点的速度轻轻推向指针，所以网格在光标停留处聚集。每个节点记住它开始时的速度，每六十分之一秒向它缓动 1%，所以指针的拉力会消失，缓慢的漂移继续。在触屏上，指针位置也来自 `pointerdown`，所以停着的手指也有效。

漂移、拉力和缓动按流逝的时间移动，而不是每帧固定量，所以 30 Hz 的手机和 144 Hz 的显示器显示相同速度。`k` 计算上一帧相当于多少个 60 Hz 帧，慢动作里是三分之一。开始、暂停或从隐藏标签页返回后的第一帧算作没有，帧之间的长间隔最多算 50ms：

```js
const dt = last === null ? 0 : Math.min(now - last, 50);   // ms since the last frame
last = now;
const k = dt / (1000 / 60) * (slow ? 1 / 3 : 1);           // 1 at 60 Hz, 2 at 30 Hz

node.x += node.vx * SPEED * k;                             // drift
node.vx += (dx / d) * 0.04 * k;                            // pull toward the pointer, when it is near
node.vx += (node.cvx - node.vx) * 0.01 * k;                // ease back to the cruising velocity
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 圆点数量 | 中等 | 少是 50 个，中等 90 个，多是 140 个；手机上最多显示 60 个，因为每帧都要检查每对圆点 |
| 连线距离 | 中等 | 两个圆点多近才会连起来：短是 90px，中等 130px，长是 180px；太长会让所有东西连到所有东西 |
| 速度 | 普通 | 圆点漂移的速度：慢速平均约每秒 8px，普通 14px，快速 23px，每个圆点有自己的随机速度和方向；更快看起来焦躁 |
| 颜色 | 青色 | 圆点和线条的颜色：青色、紫色、薄荷绿或灰色 |
| 圆点跟随指针 | 开 | 指针或手指附近的圆点朝它漂移、聚集在那里 |

## 生产注意事项
- **n² 墙**：连接测试是 O(n²)。约 150 个节点时，每帧要做 11000 多次距离检查。这个演示在手机尺寸的屏幕（宽最多 600px，或横持手机时高最多 500px）上把节点限制为 60 个，让中端手机保持 60fps。更大的场，把节点分到空间网格里，只测试相邻格子。
- **速度缓动**：每六十分之一秒，节点的速度向它开始时的速度缓动 1%。这防止指针的拉力累积成失控的速度，而且不像每六十分之一秒把速度乘以 0.99——那会在几秒内让每个节点慢到停下——它永远不会让漂移消失。把 1% 按 `k` 缩放是线性捷径，与精确的 `1 - 0.99^k` 在 50ms 的帧上最多差约 1%。
- **减少动态**：演示开始时是暂停的，显示一帧静止画面，直到访客按下播放。生产环境里，给这些访客显示静止画面。
- **视网膜屏**：为了高 DPI 屏幕上线条清晰，按 `devicePixelRatio` 缩放画布后备存储，上下文也按同样倍数缩放。这里省略了它，让手机上的填充率保持低。
- **库的等价实现**：[tsParticles](https://github.com/tsparticles/tsparticles) 用预设提供这个确切的效果（`links` 模式）；[three.js](https://threejs.org) 可以用 `LineSegments` 把同样的想法推到几万个 GPU 点。

## 另见
- [星空](../starfield/) — 显示深度而不是连线的粒子
- [漂浮元素](../floating-elements/) — 自己漂移的图形
- [网格 / 圆点图案视差](../grid-dot-pattern-parallax/) — 相对指针移动的静止网格
