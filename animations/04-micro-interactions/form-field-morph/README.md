# 表单字段变形

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/form-field-morph/)

## 它是什么
表单字段变形，也叫浮动标签，一开始每个字段的标签都在字段里面，像占位符一样。点击字段或在里面输入时，标签上移并缩小，始终可见地待在文字上方，而不是像占位符那样消失。

## 什么时候用
- 屏幕空间有限、每个输入框上方再放一个单独的 `<label>` 会太高的任何表单
- 注册、登录和结账表单
- 很多字段排成紧凑列表的设置页
- 纵向空间紧张的移动端表单

## 工作原理
标签绝对定位在字段容器里面，和占位符高度的输入框重叠。获得焦点时（或输入框填满时），它用 `translateY` 和 `scale` 向上变换：

```css
:root {
  --field-dur: 200ms;
  --float-dist: 22px;
  --label-scale: 0.8;
  --focus-color: #58a6ff;
}

.float-field { position: relative; padding-top: 20px; }

.float-field label {
  position: absolute;
  left: 0; top: 20px;
  font-size: 12px;
  color: var(--muted);
  pointer-events: none;
  transform-origin: left center;
  transition:
    transform var(--field-dur) ease,
    color var(--field-dur) ease;
}

.float-field input {
  display: block; width: 100%;
  background: transparent;
  border: none; border-bottom: 1px solid var(--border);
  outline: none;
  transition: border-color var(--field-dur) ease;
}

/* Float on focus OR when filled */
.float-field:focus-within label,
.float-field.filled label {
  transform: translateY(calc(var(--float-dist) * -1)) scale(var(--label-scale));
  color: var(--focus-color);
}
```

`.filled` 类在 JavaScript 里切换，检查输入框里是否有空格以外的文字，输入时和字段失去焦点时都检查——这样用户离开填满的字段时，标签保持浮起：

```js
function syncFilled() {
  wrapper.classList.toggle('filled', input.value.trim().length > 0);
}
input.addEventListener('input', syncFilled);
input.addEventListener('blur', syncFilled);
```

演示里，「演示一下」通过给字段加 `is-demo` 类来播放效果，而不是移动你的焦点，所以那里的焦点规则把它列在 `:focus-within` 旁边。你自己的表单里不需要它。

纯 CSS 的替代做法用 `:placeholder-shown`（占位符只在字段为空时可见）：

```css
input:not(:placeholder-shown) ~ label,
input:focus ~ label {
  transform: translateY(-22px) scale(0.8);
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 标签移动要花多长时间：慢是 320ms，普通 200ms，快 120ms；字段多的表单上慢了感觉沉重 |
| 上升多少 | 中等 | 下划线字段上标签上移多远：低是 16px，中等 22px，高是 28px；必须盖住输入的文字 |
| 升起时大小 | 中等 | 下划线字段上升起的标签有多大：小是 70%，中等 80%，大是它大小的 90% |
| 聚焦颜色 | 蓝色 | 升起的标签和活动的线条或边框的颜色 |

## 生产注意事项
- **填满状态保留的 bug**：最常见的错误是只在 `:focus-within` 上做动画。用户按 Tab 到下一个字段时，即使字段填满了，标签也会弹回去。一定要结合填满类或 `:placeholder-shown`。
- **占位符文字冲突**：浮动标签和占位符文字用途相同——不要两个都用。用浮动标签时，把 `placeholder` 属性留空，或设为一个空格 `" "`（`:placeholder-shown` 的 CSS 检测需要它）。
- **方框样式变体**：演示里有方框样式（Material 风格）的浮动标签，标签在边框里面浮动，而不是在上面。这对带边框的输入框比只用下划线的输入框更好。
- **`transform-origin: left center`**：没有它，标签会从中心缩放，水平位置会漂移。
- **无障碍**：始终用真正的 `<label>` 元素（不要只用 `aria-label` 或 `placeholder`）。读屏软件会播报标签文字；`placeholder` 文字不一定会被播报。
- **React Hook Form + Floating UI**：生产里常见的搭配。浮动状态由 `formState.dirtyFields` 或监听的字段值控制。

## 另见
- [焦点环动画](../focus-ring/) — 显示键盘在哪个项目上的圆环
- [开关滑动](../toggle-switch/) — 在开和关之间滑动的开关
- [手风琴展开/收起](../accordion/) — 平滑打开和收起的面板
- [搜索框展开](../expanding-search/) — 搜索图标打开成可以输入的字段
