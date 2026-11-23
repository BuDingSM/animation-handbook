# 复制到剪贴板

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/copy-to-clipboard/)

## 它是什么
复制反馈是「复制」按钮复制完东西后给出的简短确认。剪贴板图标变成对勾，标签从「复制」变成「已复制！」，短暂的颜色闪光标出这一刻。一两秒后按钮变回来，随时可以再用。

## 什么时候用
- 文档里的代码块和 API 密钥
- 分享链接、邀请码或优惠码
- 从表格单元格或字段里复制一个值
- 任何结果看不见、用户需要确认它确实发生了的单次点按操作

## 工作原理
两件事并行：真正的复制，和视觉状态切换。复制用的是异步的 Clipboard API，它返回一个 promise。只有它兑现后，才给按钮翻转一个 `copied` 类——CSS 靠这一个类驱动图标变形、标签交叉淡入淡出和脉冲。计时器会移除这个类，让它恢复。

```js
btn.addEventListener('click', () => {
  navigator.clipboard.writeText(snippet.innerText).then(() => {
    btn.classList.add('copied');          // icon + label + pulse
    live.textContent = 'Copied to clipboard'; // aria-live announcement
    setTimeout(() => {
      btn.classList.remove('copied');
      live.textContent = '';
    }, 1500);                              // revert delay
  });
});
```

```css
.l-copied            { opacity: 0; transform: translateY(4px); }
.copied .l-copy      { opacity: 0; transform: translateY(-4px); }
.copied .l-copied    { opacity: 1; transform: translateY(0); }
.copied .clip        { opacity: 0; transform: scale(.5); } /* clipboard out */
.copied .chk         { opacity: 1; transform: scale(1);  } /* checkmark in  */
```

两个标签叠在同一个网格单元格里，这样它们可以在原地交叉淡入淡出而不移动布局；两个图标绝对定位、叠在一起，剪贴板缩小的同时对勾放大。图标和标签动画 `opacity` 和 `transform`，对勾路径再加一段短暂的 `stroke-dashoffset` 描绘，收尾像手绘；按钮的边框和文字用颜色过渡淡成绿色，脉冲环是一次性的 `opacity` 和 `transform` 动画。如果浏览器拒绝复制，按钮保持原样，一条简短的消息会说明。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 图标和标签变化要花多长时间：慢是 450ms，普通 280ms，快 170ms |
| 变回前的时间 | 中等 | 「已复制！」停留多长时间：短是 1 秒，中等 1.5 秒，长 2.5 秒；不到一秒感觉仓促，超过大约三秒又拖得太久 |
| 手感 | 回弹 | 回弹让对勾有个小弹跳；平滑慢慢停下；均匀保持一个恒定速度 |

## 生产注意事项
- **Clipboard API 支持和回退**：`navigator.clipboard.writeText` 需要安全上下文（HTTPS 或 `localhost`）和用户手势。先做特性检测，旧浏览器或非安全上下文用隐藏的 `<textarea>` 加 `document.execCommand('copy')` 回退，演示就是这样做的。
- **不要只靠颜色**：绿色色调和脉冲是加强，不是消息本身。图标变化和“已复制！”文字传达意思，这样色盲用户也能得到清晰的反馈。
- **向读屏软件播报**：看得见的用户能看到标签变化，但辅助技术需要一个 `aria-live="polite"`（或 `role="status"`）区域，在成功时收到“已复制到剪贴板”这串文字。恢复时清空它，这样就不会重复播报过期的状态。
- **恢复时间和快速点击**：开始新的复制前，先清除待处理的计时器，否则第二次复制可能提前恢复。确认期间保持按钮可交互，这样重复复制也能用。
- **减弱动态效果**：在 `prefers-reduced-motion: reduce` 下，去掉变形、脉冲和描边绘制——立刻换图标和标签。确认仍然会到，只是不动。
- **库**：clipboard.js 从 `data-clipboard-text` 属性复制，并触发 `success` 和 `error` 事件来启动这个反馈，但它用的是较旧的 `execCommand` 路线，而不是 Clipboard API。

## 另见
- [对勾绘制](../checkmark-draw/) — 自己画出来的对勾
- [按钮按压缩放](../button-press-scale/) — 按下时会缩小的按钮
- [成功彩带](../success-confetti/) — 为更大的时刻准备的更大庆祝
- [按钮加载状态](../button-loading-states/) — 先显示旋转指示、再显示对勾或叉号的按钮
