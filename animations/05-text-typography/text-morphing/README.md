# 文字变形

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-morphing/)

## 它是什么
文字变形让一个词逐字变成下一个词。当前词的字母滑出视野，然后下一个词的字母从相反一边滑进来；字母之间的小延迟，可以让每次变化像波浪一样穿过整个词。每个词停留的时间都够读完，然后再进行下一次变化。

## 什么时候用
- 静态短语里一个在变的词：“We make [websites / apps / brands]”
- 标签改变含义时的动画状态过渡：“Loading → Ready → Error”
- 相关概念之间的主题循环：OCEAN → RIVER → STREAM
- 任何文字值会变化、而且变化应该感觉连贯而不是突兀的界面元素

## 工作原理
每个字符占一个自己的 `<span>`。从词 A 变到词 B 时，出去的 span 先拿到退场 class；等最后一个离开后，进来的 span 带着进场 class 建好，放在相反一边，拿掉它就开始向静止位置做 CSS 过渡。

`stepDur`（速度设置）和 `stepStag`（字母间延迟）在每次变化开始时读一次，慢动作下都是三倍长。延迟默认是 0，所以每个字母一起动；30 或 60ms 会让变化像波浪一样穿过整个词。`wait` 是可以被暂停按住的 `setTimeout`：

```js
// in startMorph(): the current letters leave
fromChars.forEach((c, i) => {
  c.style.transitionDelay = (stepStag * i) + 'ms';
  c.classList.add('out-' + curDir);
});
wait(afterExit, stepDur + stepStag * (fromChars.length - 1) + 60);

function afterExit() {
  wordEl.innerHTML = '';
  const enterDir = { up: 'enter-from-below', down: 'enter-from-above',
                     left: 'enter-from-right', right: 'enter-from-left' }[curDir];
  const toChars = [];
  nextWordVal.split('').forEach((c, i) => {
    const span = document.createElement('span');
    span.className = 'morph-char ' + enterDir;
    if (stepStag > 0) span.style.transitionDelay = (stepStag * i) + 'ms';
    span.textContent = c;
    wordEl.appendChild(span);
    toChars.push(span);
  });
  void wordEl.offsetWidth;   // reading the layout makes the browser apply the start position first
  toChars.forEach(c => c.classList.remove(enterDir));
  wait(afterEnter, stepDur + stepStag * (toChars.length - 1) + 50);
}
```

```css
.morph-char {
  display: inline-block;
  transition: transform var(--morph-dur) cubic-bezier(.4,0,.2,1), opacity var(--morph-dur) ease;
}

.morph-char.out-up    { transform: translateY(-100%); opacity: 0; }
.morph-char.out-down  { transform: translateY(100%);  opacity: 0; }
.morph-char.out-left  { transform: translateX(-80%);  opacity: 0; }
.morph-char.out-right { transform: translateX(80%);   opacity: 0; }

.morph-char.enter-from-below { transform: translateY(100%);  opacity: 0; }
.morph-char.enter-from-above { transform: translateY(-100%); opacity: 0; }
.morph-char.enter-from-left  { transform: translateX(-80%);  opacity: 0; }
.morph-char.enter-from-right { transform: translateX(80%);   opacity: 0; }
```

`--morph-dur` 在每次变化开始时由 `stepDur` 设定（普通速度下是 400ms）。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 滑动方向 | 上 | 上下看起来像老虎机，每个字母滑动自己的高度；左右横向滑动，字母宽度的 80%；字母在滑动时淡出淡入 |
| 字母间延迟 | 无 | 一个字母移动和下一个字母移动之间的间隔：无、短（30ms）或长（60ms）；没有延迟时每个字母一起动，有延迟会形成波浪 |
| 速度 | 普通 | 每个字母滑动需要的时间：慢速 650ms，普通 400ms，快速 250ms |
| 每个词的停留时间 | 中 | 每个词停留的时间：短 900ms，中 1500ms，长 2400ms；必须够读完 |
| 你的词语，每行一个 | OCEAN, RIVER, STORM, LIGHT, SOUND | 轮流出现的词，用大写显示 |

## 生产注意事项
- **长度不匹配**：这个演示绕开了它——整个旧词先离开，新词再进来，所以任何长度的词都行。原地换字母的变形，必须只用透明度把多余的字母淡入或淡出，这样它们周围的字母才不会跳动。
- **固定宽度的容器**：为了防止词长变化时布局位移，按最长的词给轮换词容器一个固定或 `min-width`。否则，词一换，旁边的元素就会跳动。
- **`will-change`**：演示在样式表里给字母设置了 `will-change: transform, opacity`，对一个每次变化都会重建 span 的短词来说，代价很小。屏幕上有许多字母时，在变化前加上它，变化后拿掉，因为每个被提升的 span 都会在 GPU 内存里保留自己的层。
- **GSAP**：退场用 `gsap.to(chars, { y: -50, opacity: 0, stagger: 0.03, duration: 0.4 })`，然后重建再做进场。GSAP 的 stagger 模型比 JavaScript 里逐个元素的 `transitionDelay` 更干净。
- **SplitType**：一个轻量库，把文字按词/字符拆成 `<div>` 包装，没有 GSAP 的开销。和这个模式搭配得很好。

## 另见
- [轮换词轮播](../rotate-word-carousel/) — 整个词滑动，字母不拆开
- [乱码/故障文字](../scramble-text/) — 随机符号逐个锁定为真实文字
- [打字机效果](../typewriter-effect/) — 文字逐字打出
