# 错误抖动

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/error-shake/)

## 它是什么
错误抖动告诉别人，他们输入的内容没有被接受。字段左右摆动几次，每次摆动比上一次小，最后正好回到开始的地方，像摇头说不。红色边框和一条简短消息会一起出现，说明哪里错了。

## 什么时候用
- 登录和注册表单，提交失败或为空时
- 行内字段验证（格式错误、超出范围、确认不匹配）
- 一次性密码 / PIN 输入，错误的验证码应该被拒绝，而不用跳转离开
- 任何想在字段处、而不是单独的轻提示里给出反馈的单字段拒绝

## 工作原理
字段通过切换一个 `error` 类来做动画，这个类应用一个 `@keyframes` 的 translateX 摇摆。关键帧让元素左右摆动，**幅度衰减**，然后回到 `translateX(0)`。因为只动画 `transform`，它留在合成器上，永远不会触发布局：

```css
/* the default six swings, as the demo's buildShake(6) writes them (values rounded) */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  14% { transform: translateX(calc(var(--shake-x) * -0.86)); }
  29% { transform: translateX(calc(var(--shake-x) *  0.71)); }
  43% { transform: translateX(calc(var(--shake-x) * -0.57)); }
  57% { transform: translateX(calc(var(--shake-x) *  0.43)); }
  71% { transform: translateX(calc(var(--shake-x) * -0.29)); }
  86% { transform: translateX(calc(var(--shake-x) *  0.14)); }
}
.field.error input { border-color: var(--err); animation: shake var(--shake-dur) cubic-bezier(.36,.07,.19,.97) both; }
```

演示用 JavaScript 写这条关键帧规则，所以「摆动次数」设置可以让它摆任意多次，每次比上一次小。

JS 里有一个坑：在已经有这个类的元素上重新添加它，不会重播动画。在移除和重新添加之间强制回流，浏览器才会重新开始：

```js
function shake(text){
  field.classList.remove('error','ok');
  void field.offsetWidth;        // reflow -> animation restarts
  msg.textContent = text;
  field.classList.add('error');
}
```

衰减的抖动读起来普遍像“不”，因为它*回到原点*。结束在开始的地方的动作表示拒绝——什么都没有提交。对比滑动或对勾，它们移到一个新状态，表示进展。衰减（大摆动缩小成小摆动）模仿了真实摇头和阻尼弹簧的物理感觉，所以它感觉自然，而不是像故障。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 抖动距离 | 中等 | 字段摆动多远：小是 4px，中等 8px，大是 14px，第一次摆动走完大部分，之后每次更短，直到没有；不到 4px 几乎看不见，超过约 16px 感觉激烈 |
| 摆动次数 | 中等 | 摆动多少次：少是 4 次，中等 6 次，多是 8 次；四到六次读起来像摇头，更多感觉慌张 |
| 速度 | 普通 | 整个抖动持续多长时间：慢是 650ms，普通 400ms，快 250ms；不到 250ms 感觉抽搐，超过 600ms 拖沓 |

## 生产注意事项
- **不要只靠动作或颜色。** 抖动和红色边框是加强；行内文字消息才是说明*为什么*失败的东西。色盲和减弱动态效果的用户需要文字。
- **减弱动态效果**：把抖动关进 `@media (prefers-reduced-motion: reduce)`，在那里禁用动画——红色边框加消息仍然能传达失败，不需要任何动作。
- **重新开始的技巧**：重复失败时需要回流（`void el.offsetWidth`）。也可以监听 `animationend` 再移除类，或者用 Web Animations API（`el.animate(...)`），每次调用都能干净地重新开始。
- **快速提交去抖**，这样狂按提交按钮不会堆叠动画；类切换加回流的模式已经合并成一次运行。
- **JS 层的 `prefers-reduced-motion`**：如果用 WAAPI 而不是 CSS 驱动抖动，在 JS 里检查媒体查询，设置 reduce 时跳过 `.animate()`。
- **库的等价物**：Framer Motion 把它表示为关键帧数组上的 `animate={{ x: [0,-8,8,-5,5,0] }}`；GSAP 有专门的 `RoughEase`，也可以用 `gsap.fromTo(el,{x:-8},{x:0,ease:'elastic'})`。两者都是同样的 translateX 关键帧想法，只是重新开始更方便。

## 另见
- [表单字段变形](../form-field-morph/) — 使用字段时标签从字段里升起
- [焦点环动画](../focus-ring/) — 跟随键盘焦点的圆环
- [按钮按压缩放](../button-press-scale/) — 按下时会缩小的按钮
- [对勾绘制](../checkmark-draw/) — 表示肯定的对勾
