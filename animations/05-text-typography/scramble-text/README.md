# 乱码/故障文字

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/scramble-text/)

## 它是什么
乱码文字让每个字母先显示为一个不停变化的随机字符，然后逐个锁定为真实字母。字母从左到右一个接一个地锁定，所以文字是一点一点变得可读的，就像一条隐藏信息正在被解码。随机字符可以是符号、字母表里的字母、数字、日文字符，或者混合。

## 什么时候用
- 科技产品、网络安全工具和面向开发者的网站
- 主视觉标题的戏剧性揭示，文字在页面加载时“解码”
- 导航项或互动卡片上的悬停效果
- 加载画面，在内容加载时需要一个有目的的美学

## 工作原理
每个目标字符串的字符都有自己的 `<span>`。一个 `setInterval` 循环把未锁定的 span 换成随机字符。每个字符有一个错开的 `setTimeout`，在延迟之后把它锁定为正确的值：

```js
function scramble(el, targetText) {
  const chars = '!@#$%^&*<>{}[]|/\\?=+-_~`';
  let lockedCount = 0;

  // One span per letter; each word's letters sit in one span.unit-word, which keeps the word on one line
  const spans = [];
  targetText.split(' ').filter(Boolean).forEach((word, wi, words) => {
    const w = document.createElement('span');
    w.className = 'unit-word';
    [...word].forEach(c => {
      const s = document.createElement('span');
      s.dataset.char = c;                                  // the letter it locks to
      s.textContent = chars[Math.floor(Math.random() * chars.length)];
      w.appendChild(s);
      spans.push(s);
    });
    el.appendChild(w);
    if (wi < words.length - 1) el.appendChild(document.createTextNode(' '));
  });
  const locked = new Array(spans.length).fill(false);

  // Settle each character after a staggered delay
  spans.forEach((s, i) => {
    setTimeout(() => {
      s.textContent = s.dataset.char;
      locked[i] = true;
      if (++lockedCount === spans.length) onDone();
    }, settleDelay * i + settleDelay);
  });

  // Cycle random chars on unlocked positions
  const interval = setInterval(() => {
    spans.forEach((s, i) => {
      if (!locked[i]) s.textContent = chars[Math.floor(Math.random() * chars.length)];
    });
  }, cycleMs);

  // Clear interval once all locked
  function onDone() { clearInterval(interval); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 随机字符 | 符号 | 字母闪烁经过的字符：符号看起来像黑客攻击，日文字符看起来像科幻电影里落下的代码，字母或数字看起来像密码正在被破解；混合同时从字母、数字和符号里取 |
| 字母间延迟 | 中 | 一个字母锁定和下一个字母锁定之间的等待：短是 40ms，中 70ms，长 120ms；它决定解码推进的速度 |
| 闪烁速度 | 普通 | 随机字符变化的频率：慢速每 65ms，普通每 40ms，快速每 25ms；越快看起来越混乱 |
| 你的文字 | DECODE THE MESSAGE | 要解码的文字，用大写显示 |

## 生产注意事项
- **从左到右的锁定顺序**：字符按阅读顺序锁定，词是逐步变得可读的，而不是一下子全部出现。随机或同时锁定更难读，也丢掉了“解码”的叙事。
- **字符集的选择很重要**：符号（`!@#$%`）让人想到黑客；片假名让人想到《黑客帝国》；大写字母和数字混合让人想到密码学。让字符集贴合你产品的美学。
- **不要过度使用**：每次页面切换时每个标题都乱码，会变成视觉噪音。留给主视觉时刻——一个值得戏剧性登场的词或短语。
- **大写效果更好**：小写字母带下行笔画（g、j、p、q、y），在穿过大写符号乱码时看起来别扭。大写文字更干净。
- **GSAP TextPlugin**：用可配置的 `chars` 参数处理乱码风格的效果。生产用起来 API 更简单。`gsap.to(el, { duration: 1, scrambleText: { text: "HELLO", chars: "lowerCase" } })`。
- **`aria-label` 模式**：一开始就把 `aria-label` 设为最终解码后的文字，这样屏幕阅读器播报的是正确内容，而不是乱码的中间状态。

## 另见
- [打字机效果](../typewriter-effect/) — 文字逐字打出
- [动态排版](../kinetic-typography/) — 词语逐个以各自的方式进出
- [文字变形](../text-morphing/) — 一个词逐字变成下一个词
- [翻牌器](../split-flap-board/) — 翻牌逐个拼出一个词
