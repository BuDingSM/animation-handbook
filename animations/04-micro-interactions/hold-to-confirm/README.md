# 长按确认

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/hold-to-confirm/)

## 它是什么
长按确认是一个按钮，只有按住一会儿后才会执行操作。按住时，填充会横跨它或绕着它增长；填满时，操作执行，并显示一个对勾。提前松开会让填充流回去，什么都不发生，所以一次手滑的点按或点击永远删不掉任何东西。

## 什么时候用
- 删除无法恢复的文件、项目或消息
- 关闭账号或取消订阅
- 在手机上汇款或确认付款
- 绝不能意外触发的紧急和提醒按钮
- 确认对话框太打断、但单次点按又太容易的地方

## 工作原理
一个数字 `p` 表示按钮有多满，从 0 到 1。一个帧循环按过去的时间移动它：按住时上升，松开时下降。同一个数字驱动两种样子，所以“填充样式”设置只改变显示哪一种：

```js
let p = 0, holding = false, last = null;
function frame(ts) {
  const dt = last === null ? 0 : Math.min(ts - last, 50); // elapsed time: 60 Hz and 120 Hz screens fill alike
  last = ts;
  p = holding ? Math.min(1, p + dt / holdMs) : Math.max(0, p - dt / drainMs);
  fill.style.transform = `scaleX(${p})`;   // Bar: grows from the left edge
  ring.style.strokeDashoffset = 1 - p;     // Ring: a <circle pathLength="1">, turned -90deg to start at the top
  if (p >= 1) return confirm();            // the press is used up; letting go now changes nothing
  if (holding || p > 0) requestAnimationFrame(frame);
}
```

```css
.hold { position: relative; overflow: hidden; touch-action: none; user-select: none; -webkit-touch-callout: none; }
.fill { position: absolute; inset: 0; transform: scaleX(0); transform-origin: left center; }
.ring .rf { stroke-dasharray: 1; stroke-dashoffset: 1; }
```

按住和松开来自指针和键盘。指针在 `pointerdown` 时被捕获，所以手指滑出按钮后按压还会继续；`pointerup`、`pointercancel` 和 `lostpointercapture` 都算松开。空格键和 Enter 在 `keydown` 时开始一次按住（忽略按住按键发送的重复事件），在 `keyup` 或 `blur` 时结束。在填充满之前松开，会在按钮的包装上重新启动一段短的抖动动画（这样它永远不会和按钮本身的按压缩放打架），并淡入一条“继续按住”的提示。填充满时，垃圾桶图标和标签会交叉淡出，变成对勾和“草稿已删除”，一个 `role="status"` 区域会播报；演示里一秒后所有东西又流回去，这样可以再试一次。减弱动态效果下，填充按四分之一画（`Math.floor(p * 4) / 4`），所以它是一步一步走，而不是滑动，抖动也跳过。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 按住时长 | 中等 | 按钮执行前必须按住多长时间：短是 0.8 秒，中等 1.5 秒，长是 2.5 秒；不到大约半秒，它就不再能防止手滑点按；超过三秒会感觉卡住 |
| 填充样式 | 横条 | 横条从左到右填满整个按钮；圆环绕着圆形按钮从顶部顺时针填满 |
| 提前松开会抖动 | 开 | 在填充满之前松开按钮时，短暂地左右抖动一下；“继续按住”的提示无论如何都会显示 |
| 流失速度 | 很快 | 松开时，满格的填充跑回去要花多长时间：很快是 0.4 秒，慢慢是 1.5 秒 |

## 生产注意事项
- **提供第二种方式**：有些人无法稳定地按住，有些辅助技术只发送没有按压的点击。保留同一操作的另一条路，比如确认对话框或撤销，并在按钮的标签或描述里说明它需要按住。
- **键盘**：按住按键会重复发送 `keydown`，所以忽略带 `e.repeat` 的事件，把 `keyup` 和 `blur` 当作松开，否则按住期间按 Tab 会让按钮在无人按住的情况下继续填充。
- **触摸**：`touch-action: none` 阻止浏览器把手指的一点漂移变成滚动，那会取消指针。`-webkit-touch-callout: none`、`user-select: none` 和一个调用 `preventDefault()` 的 `contextmenu` 处理函数，可以防止长按打开提示框、选中文字或显示菜单。一定要监听 `pointercancel`。
- **按时间，不按帧**：按经过的时间移动填充。每一帧固定加一点的循环，在 120 Hz 屏幕上填充速度快一倍，在掉到 30 Hz 的手机上慢一半。
- **说出发生了什么**：在 `role="status"` 的实时区域里播报结果（“旧草稿已删除”），提前松开也要播报，因为提示和抖动都是视觉上的。
- **GSAP**：`const t = gsap.to(fill, { scaleX: 1, duration: 1.5, ease: "none", paused: true, onComplete: confirm })`，然后按下时 `t.timeScale(1).play()`，松开时 `t.timeScale(4).reverse()`。
- **Framer Motion**：用 `useAnimate()`，按下时调用 `animate(fill, { scaleX: 1 }, { duration: 1.5, ease: "linear" })` 并保留它的控制器；松开时停掉它们，把 `scaleX` 动画回 0；只有在动画没有被停掉时，才在第一个动画的 `.then()` 里执行操作。

## 另见
- [按钮按压缩放](../button-press-scale/) — 按下时按钮缩小
- [进度动画](../progress-animation/) — 填满以显示进度的横条和圆环
- [错误抖动](../error-shake/) — 表示输入错了的抖动
- [对勾绘制](../checkmark-draw/) — 任务完成后自己画出来的对勾
- [按钮加载状态](../button-loading-states/) — 带旋转指示等待的按钮，然后显示对勾或叉号
