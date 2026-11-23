# 波浪文字

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/wavy-text/)

## 它是什么
波浪文字让波浪滚过一个词。每个字母都以同样的动作上下浮动，但每个字母比前一个晚一点开始，所以最高点从第一个字母走到最后一个，就像飘动的旗帜，或者一排被涌浪抬起的浮标。

## 什么时候用
- 游戏、儿童产品或休闲品牌的俏皮标题和 Logo
- 加载和空闲状态，想要一点温和的生气，又不想引人注意
- 悬停点缀，词在互动时“活过来”
- 音乐、派对或海滩主题的网站，液体感贴合语气的地方

## 工作原理
JavaScript 把字符串拆成每个字符一个 `<span>`，并给每个打上索引自定义属性 `--i`。每个 span 共享一个 `bob` 关键帧，在 Y 轴上平移；每字母的 `animation-delay` 是 `--i × stagger`，让每个字母比邻居晚一步，形成行进的波浪：

```js
[...str].forEach((ch,i)=>{
  const s=document.createElement('span');
  s.textContent=ch;
  s.style.setProperty('--i',i);
  wave.appendChild(s);
});
```

```css
.wave-text span{
  display:inline-block;
  animation:bob var(--dur) ease-in-out infinite;
  animation-delay:calc(var(--i) * var(--stagger));
}
@keyframes bob{
  0%,100%{transform:translateY(calc(var(--amp) * .6))}
  50%{transform:translateY(calc(var(--amp) * -1))}
}
```

只有 `transform` 在做动画，所以效果留在合成器上，以 60fps 运行。波浪高度、速度和字母间延迟设置这三个 CSS 变量。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 波浪高度 | 中 | 每个字母在静止位置上方升起多高：低是 8px，中 16px，高 28px，下沉是上升的 0.6 倍；越大波浪越高 |
| 速度 | 普通 | 一次上升和落下需要的时间：慢速 2.6s，普通 1.6s，快速 1s |
| 字母间延迟 | 中 | 相邻字母之间的间隔：短 30ms，中 60ms，长 120ms；延迟越长，波浪拉得越开 |
| 你的文字 | Wavy | 会波动的词，最多 14 个字母 |

## 生产注意事项
- **无障碍**：把文字拆成 span 会破坏辅助技术的可读词。演示在容器上设置 `aria-label`，在每个碎片上设置 `aria-hidden="true"`，这样屏幕阅读器会把整个词播报一次，而不是逐字母拼出来。
- **减弱动态效果**：减弱动态效果下，演示开始时是暂停的，字母保持静止，直到访客按下播放。生产中，让这些访客的字母保持静止。
- **布局**：在 span 上用 `display:inline-block`（裸 inline 元素会忽略 `transform`），在容器上用 `white-space:pre`，这样空格不会塌掉。Emoji 和组合字符会破坏简单的 `[...str]` 拆分——如果文字是用户提供的，用 `Intl.Segmenter` 分段。
- **库的等价做法**：GSAP SplitText 处理字符拆分，并在补间上提供 `stagger` 选项；Splitting.js 会发出和这里的 `--i` 等价的 `--char-index` 自定义属性，让你用纯 CSS 驱动同样的效果。

## 另见
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [打字机效果](../typewriter-effect/) — 文字逐字打出
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
- [可变字体变形](../variable-font-morph/) — 字母变字重、变倾斜
