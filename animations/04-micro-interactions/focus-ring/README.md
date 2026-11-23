# 焦点环动画

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/focus-ring/)

## 它是什么
焦点环是显示键盘在哪个项目上的轮廓。给它做动画，让圆环在每次按 Tab 时收拢到每个项目周围，每次移动都更容易跟上。它在使用键盘时显示（点进文本字段时也会显示，因为接下来要在那里打字），所以用鼠标点击按钮和链接的人不会看到圆环出现。

## 什么时候用
- 每个页面上的每个交互元素——焦点环是 WCAG 2.4.7 合规的必需项
- 键盘在字段之间导航必须清晰跟踪的表单
- Tab 顺序需要视觉确认的导航菜单和模态对话框
- 任何有依赖键盘的用户（高级用户、无障碍需求、读屏软件用户）的网站

## 工作原理
完全去掉浏览器默认的轮廓，然后用 `:focus-visible` 加动画重建它：

```css
:root {
  --ring-color: #ff9d5c;
  --ring-w: 2px;
  --ring-offset: 3px;
  --ring-dur: 140ms;
}

/* Remove default — only for elements we control */
.focusable { outline: none; }

/* Rebuild with animation — keyboard only */
.focusable:focus-visible {
  outline: var(--ring-w) solid var(--ring-color);
  outline-offset: var(--ring-offset);
  animation: ring-in var(--ring-dur) ease-out both;
}

@keyframes ring-in {
  from {
    outline-offset: calc(var(--ring-offset) + 8px);
    opacity: 0.3;
  }
  to {
    outline-offset: var(--ring-offset);
    opacity: 1;
  }
}
```

动画把 `outline-offset` 从更大的值向内插值——圆环看起来收缩到元素上，吸引眼球。辉光变体加 `box-shadow`，看起来更柔和：

```css
.focusable:focus-visible {
  outline: 2px solid var(--ring-color);
  outline-offset: 3px;
  box-shadow: 0 0 0 5px rgba(255,157,92,.18);
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 圆环样式 | 实心 | 实心是普通的轮廓；虚线用短划画出来；辉光加一圈柔和的光晕，是圆环颜色 18% 不透明度、从项目向外 5px 的带子；宽版填满项目和圆环之间的间隙，变成一条粗带 |
| 速度 | 普通 | 圆环从 8px 外收拢要花多长时间：慢是 220ms，普通 140ms，快 80ms；快到感觉不延迟，慢到能看见 |
| 圆环粗细 | 中等 | 细是 1px，中等 2px，粗是 4px；2px 或更粗容易看见 |
| 圆环间距 | 小 | 圆环和项目之间的空隙：无、3px 或 6px；2 到 4px 看起来自然 |

## 生产注意事项
- **绝不要只用 `:focus`**——它在所有浏览器里的鼠标点击时都会触发，产生设计师不想要的圆环，于是他们全局设置 `outline: none`，彻底破坏键盘访问。`:focus-visible` 同时解决了这两个问题。
- **不要全局 `outline: none`**——这会去掉所有用户的键盘导航可见性，是 WCAG 失败。只有在你自己重建圆环的元素上才去掉它。
- **浏览器支持**：`:focus-visible` 在所有现代浏览器里都支持（Chrome 86+、Firefox 85+、Safari 15.4+）。旧浏览器请用 WICG 的 `focus-visible` polyfill。
- **Framer Motion**：焦点环动画最好留在 CSS 里——JS 动画 outline-offset 性能较差。Framer Motion 的 `whileFocus` 适合 `box-shadow` 辉光变体。
- **设计令牌**：把圆环颜色公开为设计令牌。Radix UI 和 shadcn/ui 这样的系统会自动把焦点环颜色连到主题的主强调色上。

## 另见
- [表单字段变形](../form-field-morph/) — 字段获得焦点时标签上移
- [悬停状态动画](../hover-state/) — 指针在上面时项目会有反应
- [按钮按压缩放](../button-press-scale/) — 按下时按钮会缩小
