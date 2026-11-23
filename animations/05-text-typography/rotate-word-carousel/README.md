# 轮换词轮播

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/rotate-word-carousel/)

## 它是什么
轮换词让句子保持静止，只有一个词不停地换：“We craft for Designers”变成“We craft for Developers”，再变成“We craft for Humans”。当前的词滑出视野，下一个从另一边滑进来，所以每次变化都容易注意到，而句子的其余部分仍然可读。这是工作室和软件产品首页上常见的主视觉标题。

## 什么时候用
- 面向多个受众的主视觉标题：“Built for [Designers / Developers / Teams]”
- 价值主张轮换：“Faster / Simpler / Smarter”
- 服务列表：“We build [Websites / Mobile Apps / Design Systems]”
- 任何只有一个维度在变、周围语境稳定的标题

## 工作原理
轮换词放在一个 `overflow: hidden` 的容器里。当前的词向一个方向滑出，下一个从相反方向滑进来：

```html
<h1>
  <span class="static">We build for </span>
  <span class="rotating-container">
    <span class="rotating-word" aria-live="polite">Designers</span>
  </span>
</h1>
```

```css
.rotating-container {
  position: relative;
  display: inline-block;
  overflow: hidden;           /* clips words exiting above/below */
  vertical-align: middle;
}

.rotating-word {
  display: block;
  transition: transform 400ms cubic-bezier(.4,0,.2,1), opacity 400ms ease;
}

/* Exit state */
.rotating-word.out-up   { transform: translateY(-110%); opacity: 0; }

/* Enter state (transitioning TO default removes this class) */
.rotating-word.enter-below { transform: translateY(110%);  opacity: 0; }
```

```js
function transitionWord(container, nextWord) {
  const wordEl = container.querySelector('.rotating-word');
  const duration = 400;

  // Exit current
  wordEl.classList.add('out-up');

  setTimeout(() => {
    wordEl.textContent = nextWord;
    wordEl.classList.remove('out-up');
    wordEl.classList.add('enter-below');        // position below, invisible

    requestAnimationFrame(() => requestAnimationFrame(() => {
      wordEl.classList.remove('enter-below');   // animate to normal position
    }));
  }, duration + 30);
}
```

自动循环：
```js
let i = 0;
const words = ['Designers', 'Developers', 'Humans', 'Teams'];
setInterval(() => {
  transitionWord(container, words[++i % words.length]);
}, holdDuration + transitionDuration * 2);
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 滑动方向 | 上 | 上下移动词的方式像老虎机；左右是横向滑动；每次滑动都让词移动自身高度或宽度的 110%，同时淡出或淡入 |
| 速度 | 普通 | 每次滑动需要的时间，出去的词一次，进来的下一个词又一次：慢速 650ms，普通 400ms，快速 250ms |
| 每个词的停留时间 | 中 | 每个词停留的时间：短 1200ms，中 2000ms，长 3200ms；1.5 到 3 秒是可读的范围 |
| 每个词不同颜色 | 开 | 每个词换一个新颜色，会把注意力引到变化上；一个颜色更平静 |
| 你的句子 | We craft for | 标题里保持静止的部分 |
| 你的词语，每行一个 | Designers, Developers, Humans, Teams, Startups | 轮流出现的词 |

## 生产注意事项
- **只能有一个词轮换**：一个标题里多个循环的部分会造成混乱。静态语境正是让轮换词可读的原因——“We build for [X]”之所以成立，是因为“We build for”从来不变。
- **词长不一会导致布局位移**：如果词的长度不同，标题每次过渡都会重排。解决办法：（1）按最长的词给容器设一个固定的 `min-width`，（2）在词上用 `position: absolute`，手动把容器宽度设为最长词的宽度，或者（3）选长度相近的词。
- **`aria-live="polite"`**：屏幕阅读器会在当前播报完成后播报每次词的变化。设在轮换词容器上，而不是整个标题上。
- **词表保持简短**：最多 4–6 个词。再多，用户很少能看完整个循环，重复会变得麻木。稍微随机化顺序可以降低可预测性。
- **GSAP**：退场用 `gsap.to(word, { yPercent: -110, opacity: 0, duration: 0.4 })`，然后换新文字，进场用 `gsap.fromTo(word, { yPercent: 110, opacity: 0 }, { yPercent: 0, opacity: 1, duration: 0.4 })`。
- **Typed.js、Motion One**：两者都有内置的词轮换 API。Typed.js（`strings: [...]` 加 `backSpeed`）用得最广。

## 另见
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
- [入场退场排版](../enter-exit-typography/) — 整句短语进来、停留、再离开
- [打字机效果](../typewriter-effect/) — 文字逐字打出
