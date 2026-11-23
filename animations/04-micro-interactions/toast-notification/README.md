# 轻提示

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/toast-notification/)

## 它是什么
轻提示是一条小消息，来到屏幕的一角，停留几秒，然后自己离开；一条细横条倒计时它剩下的时间。几条轻提示靠得近时会堆叠，一条离开时，其他的滑过去填补空缺。点按、滑动或它的关闭按钮可以提前赶走一条。

## 什么时候用
- 确认一个后台操作完成——“已保存”“已复制”“已上传”
- 报告一个用户可以忽略或重试的非阻塞错误——“同步失败”
- 呈现用户没有明确要求的被动状态——“有新版本可用”
- 任何反馈，不应该像弹窗或对话框那样打断任务

## 工作原理
每条轻提示用 `transform` 从角落的边缘进来（滑动是 `translateX` 离屏，缩放是 `scale(.85)`，淡入淡出只有透明度），下一帧落到 `transform: none`。一个 `<span class="prog">` 运行一段 CSS `scaleX(1) → scaleX(0)` 动画，持续自动关闭的时间；它的 `animationend` 触发删除，所以在 `:hover`（或手指按住轻提示时，或键盘焦点在它里面时）暂停动画，也就免费暂停了关闭。

离开的轻提示留下的空缺用 **FLIP** 合上，而且立刻合上，而不是等退出之后。一条轻提示离开时，记录其他每条轻提示的位置，再把离开的轻提示从它站着的地方的布局里拿出来（绝对定位在它量好的地方，在其他的下面），这样它继续滑出去，但不再占地方。再读一次其他的，把每个差值作为 `translate` 播放回零——布局合上了，但只有变换在动画。单独的 `translate` 属性被使用，这样一次滑行永远不会取代一条还在滑入或被滑动的轻提示的 `transform`；`composite: 'add'` 让新的滑行可以在还在进行的一次上面开始。新轻提示先腾地方：堆叠满了，最老的在新轻提示放下之前离开，所以堆叠永远不会超过限制。在下面的角落，新轻提示放在离角落最近的地方，同样的滑行把其他的抬起来给它腾地方。

```js
const prev = new Map(toasts.map(t => [t, t.getBoundingClientRect().top]));   // First
leaving.style.position = 'absolute';        // stays where it stands (top, left and width measured before), takes no room
toasts.forEach(t => {
  const d = prev.get(t) - t.getBoundingClientRect().top;                     // First - Last
  if (!d) return;
  t.animate({ translate: [`0 ${d}px`, '0 0'] },                              // Invert, then Play
            { duration: 340, easing: 'cubic-bezier(.22,1,.36,1)', composite: 'add' });
});
```

区域是 `role="status" aria-live="polite"`，所以每条轻提示的文字会被播报，而不会抢走焦点。滑动用指针事件加 `touch-action: pan-y`，这样超过阈值的横向拖动会朝那个方向关闭，垂直页面滚动仍能通过。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 位置 | 右上 | 轻提示出现和堆叠的地方（左上、右上、左下或右下）；在下面的角落，最新的一条坐在离角落最近的地方 |
| 进入方式 | 滑入 | 滑入：从它角落的一边滑进来，从离它自己宽度稍远的地方开始；淡入；或放大：从 85% 的大小长出来，每种都在 0.34 秒内淡入；离开时用同样的方式，0.3 秒 |
| 停留时间 | 中等 | 每条轻提示停留多长时间：短是 2.5 秒，中等 4 秒，长是 6.5 秒；它的横条倒计时这段时间 |
| 同时最多 | 四个 | 两个、三个或四个；再来一条时，最老的离开 |

## 生产注意事项
- **播报，不要困住。** 把区域包在 `aria-live="polite"` 和 `role="status"` 里，读屏软件能听到消息；永远不要把焦点移进轻提示——那是对话框的行为，不是通知的行为。需要操作的错误属于提醒，不属于轻提示。
- **限制堆叠。** 无限制的堆叠会埋掉屏幕，违背目的。保持一个小的限制（3–5 条），赶走最老的，或把重复的合并成一个计数。
- **悬停和聚焦时暂停。** 一条正在被读的轻提示消失了，很不友好。当关闭由 `animationend` 驱动时，在 `:hover`/`:focus-within` 上暂停进度动画，也就暂停了计时器。
- **遵循减弱动态效果。** 在 `prefers-reduced-motion: reduce` 下，去掉滑动/缩放；立刻出现和消失，隐藏横条但让它的倒计时在看不见的地方继续跑，这样 `animationend` 仍会关闭轻提示，悬停和聚焦仍会暂停它。
- **用变换回流，永远不要用布局。** 动画 `top`/`height` 来合上空缺会搅动布局；FLIP 把运动留在 GPU 友好的 `transform` 上。
- **库的等价物**：[Sonner](https://sonner.emilkowal.ski/) 和 `react-hot-toast` 开箱即用提供堆叠、滑动和悬停暂停；Radix Toast 为自定义皮肤提供无障碍原语（实时区域、滑动、计时器）。

## 另见
- [抽屉滑出](../drawer-slide/) — 从边缘滑入的更大面板
- [弹窗展开](../modal-expand/) — 继续之前必须关掉的窗口
- [成功彩带](../success-confetti/) — 为完成的任务庆祝
- [灵动岛](../dynamic-island/) — 长成实时通知、再缩回去的黑色药丸
