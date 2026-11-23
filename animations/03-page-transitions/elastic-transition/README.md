# 弹性过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/elastic-transition/)

## 它是什么
弹性过渡让新页面滑进来时冲过自己的位置一点，再弹回来落定，而不是减速到停。过冲让运动有了分量，就像页面是被橡皮筋拉到位的一样。弹跳可以提前规划成一条固定路径，成本低；也可以通过模拟弹簧实时算出来，感觉更真实。

## 什么时候用
- 活泼、有个性的界面，动效本身就是个性的一部分
- 滑入的面板、卡片和页面，应该感觉真实而不是机械
- 确认或抵达的时刻，一个小弹跳表示「落定了」
- 手势可能被打断的情况——实时弹簧能对打断做出反应，关键帧做不到

## 工作原理
这个演示提供两种弹跳方式。预设路径在运行时写一条 `@keyframes` 规则，中间的百分比先推过终点再回来，过冲量由弹跳幅度设置。实时弹簧以 1/120 秒的固定步长积分一个真实的弹簧：力等于 `-刚度 × 位移 − 阻尼 × 速度`，每一帧按经过的时间需要多少步就走多少步，循环一直跑到位置和速度都低于阈值：

```js
const STEP = 1 / 120;                            // the spring moves in steps of 1/120 of a second
function doSpring(o, n, stiffness, damping) {    // o: the old page, n: the new page
  let pos = 100, vel = 0, target = 0;            // start off-screen right
  let before = pos, acc = 0, lastTime = null;    // acc: time the steps have yet to cover
  function tick(now) {
    if (lastTime !== null) acc += Math.min((now - lastTime) / 1000, 0.05);
    lastTime = now;
    let settled = false;
    for (let i = 0; acc > 0 && i < 8 && !settled; i++) {   // at most 8 steps in a frame
      const f = -stiffness * (pos - target) - damping * vel;  // Hooke's law + damping
      before = pos;
      vel += f * STEP;
      pos += vel * STEP;
      acc -= STEP;
      settled = Math.abs(pos - target) < 0.5 && Math.abs(vel) < 0.5;
    }
    if (acc > 0) acc = 0;                        // step limit reached: drop the time left
    if (settled) {
      n.style.transform = '';                    // settled — clean up
    } else {
      const shown = pos + (before - pos) * (-acc / STEP);     // the place at this frame's own time
      n.style.transform = `translateX(${shown.toFixed(2)}%)`;
      requestAnimationFrame(tick);
    }
  }
  requestAnimationFrame(tick);
}
```

一帧把自上一帧以来经过的时间加到 `acc` 里，绝不超过 50ms，这样弹簧在暂停、慢帧或后台标签页之后不会跳到前面，再把它花在整数步上。第一帧不加时间，因为没有更早的帧可供测量。最后一步可能在这一帧之后结束一点，所以页面画在最后两步之间、这一帧自己的时间上；一帧里装不下整数步时页面仍然会动。在演示页上，速度和弹跳幅度同时设置两种弹跳方式：预设路径的长度（1400、900 或 550ms）和过冲（30、60 或 100），或实时弹簧的刚度（75、200 或 640）和阻尼。这些值是跑这个循环测出来的，所以弹簧冲过自己位置的距离与预设路径差不多（约页面宽度的 3.6%、7.2% 和 12%）。按整数步推进弹簧比通常的弹簧公式预期的更平静，所以这些值是测出来的，而不是算出来的。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 弹跳方式 | 预设路径 | 预设路径播放一条事先定好的弹跳，成本低；实时弹簧以小的固定时间步算出运动，感觉真实，还能对打断做出反应 |
| 弹跳幅度 | 中 | 页面在落定前冲过自己位置多远：预设路径上小是页面宽度的 3.6%，中 7.2%，大 12%，实时弹簧也差不多冲那么远；弹跳越大意味着阻尼越小，所以摆动时间更长 |
| 速度 | 普通 | 多快落定：预设路径上慢是 1400ms，普通 900ms，快 550ms；用实时弹簧时，更快的速度就是更硬的弹簧，慢时约 1 到 1.3 秒落定，普通时 0.6 到 0.8 秒，快时 0.3 到 0.45 秒 |

## 生产注意事项
- **刚度和阻尼互相影响。**低于临界阻尼（`damping < 2√stiffness`）时弹簧会振荡；达到或超过它时，它会不弹跳地缓入。把这对值一起调——提高刚度通常需要更多阻尼才能保持得体。
- **限制一帧增加的时间。**直接用原始帧间隔走一步是经典的弹簧 bug：一帧很长之后 `pos` 会冲向无穷大。固定步长避免了这一点，但一帧增加的时间仍然需要限制：整个加进去的话，暂停或慢帧会让弹簧跳到前面，弹跳在一帧内就结束了。50ms 的限制（代码片段里的 `0.05` 秒）让它从原来的地方继续动，也让一帧只有少数几步。
- **帧率。**演示按经过的时间需要多少步、就以 1/120 秒的固定步长移动弹簧，所以它在任何帧率下弹跳都一样。按每帧一步的弹簧不是这样：在每秒 30 帧时小弹跳几乎看不见，在 120 帧时则稍大一点。在最后两步之间画页面，否则装不下整数步的一帧会让弹簧看起来静止不动。
- **CSS 关键帧过冲可能被裁。**如果页面在弹跳完成前完全平移到边缘，过冲会把内容滑出可见范围。留一点余量，或在动画期间让容器溢出。
- **等价的库。**Framer Motion 和 React Spring 直接接受 `stiffness`/`damping`，还帮你处理打断——生产环境用它们，而不是手写积分器。GSAP 的 `elastic.out` 缓动近似 CSS 关键帧的感觉。CSS 的 `linear()` 缓动现在无需 JavaScript 就能编码一条采样过的弹簧曲线。

## 另见
- [滑动过渡](../slide-transition/) — 它给普通滑动加上弹簧
- [缩放过渡](../zoom-transition/) — 页面在纵深中移动而不是滑动
- [FLIP 技巧](../flip-technique/) — 元素滑到新位置
- [变形过渡](../morph-transition/) — 页面之间形状变化
