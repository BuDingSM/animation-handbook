# 按钮加载状态

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/button-loading-states/)

## 它是什么
按钮加载状态在按钮自己内部显示：按下后正在处理。标签让位给旋转指示，按钮不能被按两次，收到答复后，一个对勾或叉号先说结果怎么样，标签再回来。按钮也可以缩成旋转指示周围的圆形，用裁剪画出来，周围什么都不会动。

## 什么时候用
- 等待服务器的保存、发送和提交按钮
- 联系、反馈和注册表单
- 结账和支付按钮，按两次会重复下单
- 任何耗时大约半秒到几秒的操作

## 工作原理
按钮在布局里始终保持完整尺寸。访客看到的形状是它里面的一个颜色层，状态是按钮上的类：`busy`，然后是 `ok` 或 `err`，缩小时还有 `round`。

```css
.bg { position: absolute; inset: 0; border-radius: 26px; background-color: #ff9d5c;
      clip-path: inset(0 0 round 26px);
      transition: clip-path 300ms cubic-bezier(.22, 1, .36, 1), background-color 300ms; }
.round .bg { clip-path: inset(0 calc(50% - 26px) round 26px); }   /* a 52px circle in the middle */
.ok .bg  { background-color: #3fb950; }
.err .bg { background-color: #f85149; }
.busy .spin { opacity: 1; animation: spin .7s linear infinite; }  /* turns only while busy */
.ok .tick path, .err .cross path { stroke-dashoffset: 0; }        /* paths with pathLength="1", dasharray 1 */
```

```js
function submit() {
  if (state !== 'rest') return;                     // a second press does nothing
  state = 'busy';
  btn.setAttribute('aria-disabled', 'true');        // not disabled: the keyboard focus stays on the button
  btn.classList.add('busy');
  btn.classList.toggle('round', shrinks && !reducedMotion);
  status.textContent = 'Sending…';
  send().then(ok => {                               // the demo waits How long it loads instead
    btn.classList.replace('busy', ok ? 'ok' : 'err');
    status.textContent = ok ? 'Feedback sent.' : 'Not sent. Please try again.';
    setTimeout(() => btn.classList.remove('ok', 'err', 'round'), 1300); // the change (300ms), then a second to read it
  });
}
```

标签和旋转指示在原地交叉淡入淡出，对勾和叉号沿着自己的长度画出来，出错时按钮外框会短暂地抖一下。去掉这些类，一切都会播回「提交」；只有播完后按钮才能再次接受按压。焦点环在按钮很宽时在按钮外面，变圆时移到圆圈周围。减弱动态效果下按钮保持形状，什么都不转、不抖，变化的是标签的文字：发送中…、✓ 已发送或 ✕ 未发送。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 结果 | 成功 | 成功在绿色上显示对勾；失败在红色上显示叉号，并让按钮轻轻抖一下，在 0.4 秒内向两侧摆动 7、6、4、2px |
| 速度 | 普通 | 形状、颜色和图标变化要花多长时间：慢是 500ms，普通 300ms，快 180ms |
| 缩成圆形 | 开 | 开：把按钮的颜色裁剪成旋转指示周围的圆形；关：保持完整形状，只改变里面的内容 |
| 加载时长 | 中等 | 按钮变好形状后，旋转指示转多长时间、答案才到来：短是 0.7 秒，中等 1.2 秒，长 2.2 秒；真实表单里这是服务器的时间 |

## 生产注意事项
- **用 `aria-disabled`，不用 `disabled`**：按钮在有焦点时变成 `disabled`，焦点会掉到页面上，键盘用户会丢了位置。让它保持可聚焦，标记 `aria-disabled="true"`，并在处理函数里忽略按压。
- **播报出来**：旋转指示和图标只是视觉上的。一个说“发送中…”然后说结果的 `role="status"` 区域，能告诉读屏软件用户发生了什么。
- **不要闪烁**：如果大多数答复都在大约 300ms 内回来，等过这个延迟再开始加载的样子，否则每次快速按压都会闪一下。
- **说清失败原因**：叉号只说了失败，没说为什么。在表单附近留一条消息，说明访客能修复的错误，并把他们输入的内容留在原地。
- **长时间等待**：超过几秒，按钮里的旋转指示会让人觉得卡住了。上传和其他长任务请用进度条。
- **绝不要给宽度做动画**：按钮在布局里的位置保持大小，所以它缩小时表单不会跳动；变的只有裁剪和颜色层。
- **GSAP**：`gsap.to(bg, { clipPath: "inset(0% 41% 0% 41% round 26px)", duration: 0.3, ease: "power3.out" })`，起始裁剪用同样的百分比写。
- **Framer Motion**：在颜色层上用 `<motion.span animate={{ clipPath: busy ? "inset(0% 41% 0% 41% round 26px)" : "inset(0% 0% 0% 0% round 26px)" }} />`，并用 `AnimatePresence` 切换标签、旋转指示和图标。

## 另见
- [对勾绘制](../checkmark-draw/) — 任务成功后自己画出来的对勾
- [加载旋转指示](../loading-spinner/) — 六个小图形，表示有东西在加载
- [错误抖动](../error-shake/) — 表示输入错了的抖动
- [按钮按压缩放](../button-press-scale/) — 按下时按钮会缩小
- [长按确认](../hold-to-confirm/) — 按住会填满、满了才执行的按钮
