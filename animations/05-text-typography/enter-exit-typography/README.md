# 入场退场排版

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/enter-exit-typography/)

## 它是什么
入场退场排版在同一个地方一次展示一句短语。每句短语进来、静止停留到够读完，然后在下一句到来之前离开。每句短语都有自己的进出方式，比如滑动、放大、模糊或擦除，序列结束时最后一句留在屏幕上。

## 什么时候用
- 连续的品牌宣言和宣言式段落
- 自动循环、无需用户互动的幻灯片式内容
- 一步一步讲故事的应用引导流程
- 氛围展示屏和数字标牌
- 带有轮换信息队列的主视觉区域

## 工作原理
每句短语有一个三计时器链：进场动画触发，然后在进场时长之后停留计时器开始，再在停留时长之后退场动画触发，最后在退场时长之后调用 `done()` 进入下一句短语：

```js
function show(phrase, done) {
  el.textContent = phrase.text;
  el.classList.remove(...allClasses, 'hidden');
  el.classList.add(phrase.enter);           // triggers CSS @keyframes

  setTimeout(() => {                         // after enter completes
    el.classList.remove(phrase.enter);
    el.classList.add(phrase.exit);          // triggers exit @keyframes

    setTimeout(() => {                       // after exit completes
      el.classList.add('hidden');
      done();                               // advance to next phrase
    }, EXIT_DURATION + 50);
  }, ENTER_DURATION + HOLD_DURATION);
}
```

每对进场/退场都是一条 `@keyframes` 规则：

```css
.enter-slide-right { animation: eSR 500ms ease-out forwards; }
.exit-slide-left   { animation: xSL 400ms ease-in  forwards; }

@keyframes eSR { from { opacity:0; transform:translateX(-50px) } to { opacity:1; transform:translateX(0) } }
@keyframes xSL { from { opacity:1; transform:translateX(0) }    to { opacity:0; transform:translateX(60px) } }
```

一次播放按顺序展示短语，最后一句留在屏幕上。循环开启时，最后一句也会退场，播放从第一句重新开始。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 进场速度 | 普通 | 每句短语进来需要的时间：慢速 800ms，普通 500ms，快速 300ms；快速感觉有活力，慢速感觉郑重 |
| 每句短语的停留时间 | 中 | 每句短语静止停留的时间：短 1200ms，中 2000ms，长 3200ms；必须比读完这句短语需要的时间长 |
| 退场速度 | 普通 | 每句短语离开需要的时间：慢速 650ms，普通 400ms，快速 250ms；退场通常比进场短 |

## 生产注意事项
- **停留时间是最重要的参数**：如果短语没有足够的时间被读完，动画就白做了。一句 5 个词的短语至少需要 1000ms 的停留，最好 1500–2000ms。用舒服的速度把短语大声读出来测试——如果在退场开始前读不完，就延长停留。
- **方向连续性**：如果一句短语向上退场，下一句从下方进来，会让人感觉在同一条垂直轨道上。如果一句短语向左退场，下一句从右边进来，就暗示用户正在“向前”通过一个序列。这些都是电影剪辑里的惯例。
- **标签失焦时自动暂停**：用 `document.addEventListener('visibilitychange', ...)` 在标签失去焦点时暂停序列。用户不在时还在推进的短语，会导致信息被错过。
- **`prefers-reduced-motion`**：把所有动作换成即时切换——`display: none` → `display: block`。文字序列仍然循环，只是没有动画。
- **Framer Motion**：`<AnimatePresence>` 优雅地处理 React 组件的进场/退场生命周期。每句短语按条件渲染，并拿到 `initial`、`animate` 和 `exit` 属性。

## 另见
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [轮换词轮播](../rotate-word-carousel/) — 句子中的一个词不断换成下一个
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
