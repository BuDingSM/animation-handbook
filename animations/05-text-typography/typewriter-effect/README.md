# 打字机效果

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/typewriter-effect/)

## 它是什么
打字机效果让文字像有人在打字一样一个字符一个字符地出现，最后一个字符后面跟着一个闪烁的光标。每次按键之间时间上的微小随机差异，让它感觉像是人打出来的，而不是机器，因为真实的打字从来不是完全均匀的。

## 什么时候用
- 开发者工具、终端主题或黑客美学网站上的主视觉标题
- 打出来比一下子出现更好的代码示例
- 打字逐步揭示信息的简短标语和行动号召
- 聊天界面、AI 演示和流式文本界面

## 工作原理
核心循环是数数，并把文字截到那个长度显示出来。设置 `textContent` 会替换元素里的所有内容，包括光标 `<span>`，所以每一步都会重新加上光标，它总是在最后一个字符后面。`later` 是会记住计时器 id 的 `setTimeout`，这样重播可以取消还在等待的按键：

```js
function render(shown) {
  typeOut.textContent = shown;
  typeOut.appendChild(cursor);
}

function typeChars(text, done) {
  let i = 0;
  (function tick() {
    render(text.slice(0, i));
    if (i < text.length) { i++; later(tick, jitter(spd)); }
    else done();
  })();
}

// Types like a person: up to 20ms earlier or later; Slow motion (mul() = 3) stretches every delay
function jitter(base) {
  const varied = jitterTog.checked ? base + Math.round((Math.random() - 0.5) * 40) : base;
  return varied * mul();
}
```

光标通过 CSS 动画闪烁；竖线形状是光标颜色的 2px 宽的竖条：

```css
.cursor {
  display: inline-block;
  width: 3px;
  height: 1.1em;
  vertical-align: text-bottom;
  margin-left: 2px;
  animation: blink 1s step-end infinite;
}
.cursor.pipe { width: 2px; background: var(--cursor-color); }
@keyframes blink {
  0%, 100% { opacity: 1; }
  50%      { opacity: 0; }
}
```

对于删除并重打的变体，索引改为倒数，删掉最后一个词和它前面的空格（从来不超过文字的最后 40%）。然后 `retypeChars` 把删掉的词重新打出来，后面加上 " — always."：

```js
function deleteChars(text) {
  const words = text.trim().split(' ');
  const deleteFrom = Math.max(text.length - words[words.length - 1].length - 1, Math.floor(text.length * 0.6));
  let i = text.length;
  (function del() {
    if (i > deleteFrom) { i--; render(text.slice(0, i)); later(del, jitter(30)); }
    else retypeChars(text, deleteFrom);
  })();
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 每次按键之间的时间：慢速是 80ms，普通 50ms，快速 30ms；大约 80 到 120ms 感觉像是自然的打字节奏 |
| 光标形状 | 竖线 | 方块读起来像终端，竖线像文本编辑器，下划线像老式电脑屏幕 |
| 删除并重打 | 关 | 把文字打出来，倒回去删掉最后一个词（每个字符大约 30ms），然后用一个新的结尾重新打出来 |
| 像真人一样打字 | 开 | 让每次按键最多提前或推后 20ms，节奏不均匀，像真人打字 |
| 光标颜色 | 粉色 | 光标的颜色；选一个和文字形成对比的颜色 |
| 你的文字 | 两句示例句子 | 要打出来的文字；短行效果最好 |

## 生产注意事项
- **长文字让人疲惫**：打字机效果适合简短的主视觉文案（大约 15 个词以内）。整段文字一个字符一个字符地打出来，会逼着用户等待他们本来能立刻读完的内容。把它留给戏剧性的揭示，而不是正文。
- **屏幕阅读器**：它们应该只听一次完整文字，而不是每个字符打出来时都听一次。不要在正在打字的元素上放 `aria-live`（它会播报每次按键）；把正在打字的元素用 `aria-hidden="true"` 藏起来，给它的容器 `role="img"` 和一个放着完整最终文字的 `aria-label`，就像这个演示做的那样。
- **循环和轮换**：“打出来、删掉、重打”的循环（在多个短语之间轮换）是最常见的生产模式。每个短语打出来，短暂停留，然后在下一个开始前删掉。
- **性能**：`setTimeout` 的精度够用了；`requestAnimationFrame` 对打字计时来说是多余的，还会带来不必要的复杂性。
- **Typed.js**：这个效果的经典库。处理多个字符串、删除、循环、智能删除（只删不同的后缀），以及字符串里的 HTML 标签。生产中值得用它，而不是自己写。

## 另见
- [乱码/故障文字](../scramble-text/) — 随机符号逐个锁定为真实文字
- [轮换词轮播](../rotate-word-carousel/) — 句子中的一个词不断换成下一个
- [入场退场排版](../enter-exit-typography/) — 每句短语进来、停留足够读完、再离开
