# 对勾绘制

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/checkmark-draw/)

## 它是什么
自己画出来的对勾，像手绘一样，确认某件事成功了。圆圈和对勾从一开始就在那里，只是藏起来了，沿着长度一点点露出来，所以线条看起来像被画出来一样。同样的技巧可以在失败时画一个红叉号。

## 什么时候用
- 表单提交成功状态（按钮变成对勾）
- 支付确认界面
- 多步流程里的步骤完成指示
- 任何适合用视觉奖励的二元成功/失败状态

## 工作原理
每条 SVG 路径都有一个 `getTotalLength()` 值——它描边的像素长度。把 `stroke-dasharray` 设为这个长度、`stroke-dashoffset` 也设为同样长度，描边就看不见了（缺口把它完全盖住了）。把 `stroke-dashoffset` 动画到 0，就把路径画出来了：

```html
<svg viewBox="0 0 80 80">
  <circle class="check-circle" cx="40" cy="40" r="30"/>
  <polyline class="check-mark" points="24,41 35,52 56,30"/>
</svg>
```

```css
.check-circle {
  fill: none; stroke: #56d364; stroke-width: 4;
  stroke-dasharray: 189; /* 2π × 30, rounded up */
  stroke-dashoffset: 189;
  transition: stroke-dashoffset 500ms ease-out;
}
.check-mark {
  fill: none; stroke: #56d364; stroke-width: 4; stroke-linecap: round; stroke-linejoin: round;
  stroke-dasharray: 52;
  stroke-dashoffset: 52;
  transition: stroke-dashoffset 500ms ease-out;
}

.drawn .check-circle { stroke-dashoffset: 0; }
.drawn .check-mark   { stroke-dashoffset: 0; transition-delay: 200ms; } /* delay after circle */
```

圆圈先画（0ms 延迟），对勾带着 `200ms` 延迟跟上——就形成了“先画圆圈、再画对勾”的顺序效果。延迟在 `.drawn` 规则上，所以只在向内画的时候生效；去掉这个类时，两条线会一起画回去。演示把延迟设为画线时间的 0.4 倍，所以它会跟着「速度」设置走。

对于不是简单几何图形的路径，用 JavaScript 测量长度：

```js
const path = document.querySelector('.my-path');
const length = path.getTotalLength();
path.style.strokeDasharray = length;
path.style.strokeDashoffset = length;
// Trigger draw:
path.style.strokeDashoffset = 0;
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 结果 | 成功 | 成功画一个圆圈和对勾，并把按钮变成绿色；失败画一个红叉号，抖一下（0.4 秒内向两侧摆动 6、6、4、4px），并把按钮变成红色 |
| 速度 | 普通 | 每条线画出来要花多长时间：慢是 800ms，普通 500ms，快 300ms，对勾在圆圈之后 40% 的时间开始（叉号的第二笔在第一笔之后 30%）；200 到 800ms 很容易跟上 |
| 线条粗细 | 中等 | 细是 2px，中等 4px，粗是 7px，图标 80px 宽，圆圈 60px 宽；粗显得自信，细显得精确 |
| 手感 | 平滑 | 平滑在结尾放慢，像笔抬起来一样；轻柔是缓入缓出；均匀以一个恒定速度画 |
| 对勾颜色 | 绿色 | 圆圈和对勾的颜色；叉号保持红色 |

## 生产注意事项
- **路径长度必须和 `stroke-dasharray` 一致**：长度错了，路径会画不全或画过头。对于计算出来的路径，一定要用 `getTotalLength()`，而不是硬编码。
- **`<polyline>` 上的 `stroke-dasharray`**：多段线在所有主流浏览器里 `getTotalLength()` 都报告正确。对勾就用它；`<path d="M 24,41 L 35,52 L 56,30">` 是等价的。
- **失败状态配对**：同样的技巧可以给失败画一个叉号。两条交叉的 `<line>` 元素加上错开的延迟，就能做出交叉画出的效果（演示的「结果」设置展示了它）。
- **出错时抖动**：画完叉号后，在容器上用 `animation: shake 400ms ease`，能加强拒绝的感觉。
- **GSAP**：`gsap.to(path, { strokeDashoffset: 0, duration: 0.5, ease: "power2.out" })`。DrawSVG 插件对复杂路径更方便。
- **Framer Motion**：`<motion.path initial={{ pathLength: 0 }} animate={{ pathLength: 1 }} transition={{ duration: 0.5 }} />`——`pathLength` 是 dashoffset 技巧的 0–1 简写。

## 另见
- [进度动画](../progress-animation/) — 在成功之前显示完成了多少
- [加载旋转指示](../loading-spinner/) — 对勾之前的等待标志
- [按钮按压缩放](../button-press-scale/) — 按下时按钮会缩小
