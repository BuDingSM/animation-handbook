# 开关滑动

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/toggle-switch/)

## 它是什么
开关滑动是一个开/关控件，圆形旋钮滑过轨道，打开时轨道填满颜色。滑动展示了变化，而不是只翻转它；它的速度和曲线决定开关感觉机械、有分量还是有弹性。

## 什么时候用
- 设置和偏好面板（通知开/关、深色模式、功能开关）
- 任何二元选项，实体开关的视觉隐喻有助于理解
- 以移动为主的界面，开关比复选框更容易点按
- 表格行或列表项里的内联开关

## 工作原理
开关完全用 CSS 构建，基于一个隐藏的 `<input type="checkbox">`。轨道背景和滑块位置都在 `:checked` 上过渡：

```html
<label class="sw">
  <input type="checkbox">
  <span class="track"></span>
  <span class="thumb"></span>
</label>
```

```css
:root {
  --tog-dur: 200ms;
  --tog-ease: cubic-bezier(.4, 0, .2, 1);
  --on-color: #58a6ff;
}

.sw { position: relative; display: inline-block; width: 52px; height: 30px; }
.sw input { position: absolute; opacity: 0; width: 0; height: 0; }

.track {
  position: absolute; inset: 0; border-radius: 15px;
  background: #21262d;
  transition: background var(--tog-dur) var(--tog-ease);
}
.thumb {
  position: absolute; top: 3px; left: 3px;
  width: 24px; height: 24px; border-radius: 50%; background: #fff;
  transition: transform var(--tog-dur) var(--tog-ease);
  box-shadow: 0 1px 4px rgba(0,0,0,.4);
}

.sw input:checked ~ .track { background: var(--on-color); }
.sw input:checked ~ .thumb { transform: translateX(22px); }
.sw input:focus-visible ~ .track { outline: 2px solid #ff9d5c; outline-offset: 2px; }
```

复选框本身是看不见的，所以键盘焦点画在它旁边的轨道上；没有最后那条规则，用 Tab 到开关的人会什么都看不到。

想要有弹性的感觉，把滑块过渡的缓动换成 `cubic-bezier(.34, 1.56, .64, 1)`——这会在落定前造成一点过头。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 旋钮滑过去要花多长时间：慢是 320ms，普通 200ms，快是 120ms；不到 120ms 感觉机械，超过 350ms 迟钝 |
| 手感 | 轻柔 | 轻柔缓入缓出，让旋钮有分量；平滑开始快、慢慢停下；均匀以一个稳定的速度滑动 |
| 回弹旋钮 | 开 | 第三个开关的旋钮会稍微超过末端，再落定 |
| 开启颜色 | 蓝色 | 开关打开时轨道的颜色；它必须和白色旋钮对比明显。“外观”开关保持它自己的橙色 |

## 生产注意事项
- **无障碍**：隐藏的 `<input type="checkbox">` 免费提供键盘控制、`aria-checked` 状态和读屏软件播报——状态管理不需要 JS。
- **标签包裹**：把整个组件包在 `<label>` 里，让整个表面（轨道 + 滑块）都成为点击/点按目标——不需要 `for`/`id` 接线。演示的开关名字也是标签：每个名字都是一个指向它复选框的 `<label for>`，所以点按名字会翻转开关，读屏软件播报的也是它。
- **`touch-action: manipulation`**：把它加到标签上，消除移动浏览器上 300ms 的延迟。
- **弹性缓动陷阱**：`cubic-bezier(.34,1.56,.64,1)` 里的弹簧过头，在弹跳完成前滑块到达轨道边缘时，视觉上会被裁剪。加 1–2px 的额外轨道内边距来补偿。
- **Headless UI / Radix UI**：`<Switch>` 组件处理所有 ARIA 和键盘事件。通过 `data-state="checked"` 属性做样式。
- **Framer Motion**：在 0 和轨道宽度之间动画一个 `<motion.div>` 的 `x` 值——对受控组件来说比 CSS 更简单。

## 另见
- [按钮按压缩放](../button-press-scale/) — 按下时按钮缩小
- [表单字段变形](../form-field-morph/) — 字段聚焦时标签上移
- [手风琴展开/收起](../accordion/) — 平滑打开和关闭的面板
