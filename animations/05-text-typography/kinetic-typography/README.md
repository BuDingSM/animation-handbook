# 动态排版

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/kinetic-typography/)

## 它是什么
动态排版是会动的文字，每个词的动作都贴合它的意思。一个短序列一次展示一句短语，每句都有自己的进场和退场方式：重的词可以大大地落进来，轻的词可以轻轻淡入。它源自电影片头和动态图形，在那里，会动的文字自己就能讲故事。

## 什么时候用
- 开场序列和启动画面，在那里确立品牌个性
- 营销网站上像视频一样的讲故事段落
- 需要有动态图形感觉的演示幻灯片
- 应用引导流程，每一步都有一个叙事节拍
- 任何文字本身就是内容、而不是别的东西的标签的地方

## 工作原理
每句短语都是一个 CSS 元素，有进场动画、停留期和退场动画。JavaScript 的时间线用 `setTimeout` 把它们串起来：

```js
const PHRASES = [
  { text: 'EVERY',  enter: 'enter-scale', exit: 'exit-up',    hold: 1400 },
  { text: 'MOMENT', enter: 'enter-big',   exit: 'exit-shrink', hold: 1800 },
];

function showPhrase(i) {
  const p = PHRASES[i];
  el.textContent = p.text;
  el.className = p.enter; // triggers @keyframes
}

function run(idx = 0) {
  showPhrase(idx);
  const last = idx === PHRASES.length - 1;
  if (last && !loop) return;               // the final phrase stays on screen
  setTimeout(() => {
    exitPhrase(idx, () => run((idx + 1) % PHRASES.length));
  }, ENTER_DUR + PHRASES[idx].hold);
}
```

每个进场和退场都是一条独立的 `@keyframes` 规则：

```css
.enter-scale { animation: kEnterScale 600ms ease-out forwards; }
.exit-up     { animation: kExitUp     400ms ease-in  forwards; }

@keyframes kEnterScale { from { opacity:0; transform:scale(.5) } to { opacity:1; transform:scale(1) } }
@keyframes kExitUp     { from { opacity:1; transform:translateY(0) } to { opacity:0; transform:translateY(-50px) } }
```

速度设置是一个 CSS 自定义属性，用在每个 `calc()` 里；计时器除以同一个值：

```css
.enter-scale { animation-duration: calc(600ms / var(--spd)); }
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 整个序列的节奏：慢速是 0.6 倍速，普通是 1 倍速，快速是 1.7 倍速；进场、停留和退场一起缩放，普通速度下进场需要 400 到 700ms，停留 0.9 到 2.2 秒，退场 300 到 500ms |

## 生产注意事项
- **先有文案，再编动作**：动作要为词语而选，而不是反过来。轻柔的词消散，沉重的词砸进来，脆弱的词颤动。统一套用的通用进场和退场会毁掉整个手法。
- **停留时间至关重要**：字幕的阅读速度大约是每个词最少 200ms。一句 3 个词的短语至少需要 600ms 的停留，最好 1000ms 以上。最常见的错误是停留太短。
- **音频同步**：动态排版起源于与声音同步的广播。即使在网页上（没有音频），把短语排到心里的节拍上，也会让序列感觉更有意图。用小节来想，而不只是毫秒。
- **CSS 还是 Canvas**：CSS `@keyframes` 对大多数网页用途来说够用了。对于需要与音频同步或非常复杂的序列，GSAP 的时间线 API（`gsap.timeline()`）是生产级的选择。Lottie（基于 JSON 的 After Effects 导出）处理那些用 CSS 做起来太复杂的预渲染动态序列。
- **`prefers-reduced-motion`**：同时显示所有短语，或者只显示最后一句——不要把文字本身藏起来。

## 另见
- [入场退场排版](../enter-exit-typography/) — 每句短语进来、停留、再离开
- [轮换词轮播](../rotate-word-carousel/) — 句子中的一个词不断换成下一个
- [裁剪揭示文字](../text-clip-path-reveal/) — 标题的每一行依次被揭开
