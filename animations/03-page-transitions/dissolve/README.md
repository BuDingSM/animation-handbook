# 溶解过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/dissolve/)

## 它是什么
溶解把一次页面切换打碎成一格瓷砖。每块瓷砖在略有不同的时刻淡入、盖住旧页面，所以旧页面看起来像是碎成颗粒或方块，而不是均匀淡出。瓷砖的顺序——随机、对角线扫过或从中间向外成环——决定了溶解的质感。

## 什么时候用
- 想要比普通交叉淡入淡出更有质感、但比滑动或缩放不那么夸张的过渡
- 照片和视频场景，电影风格的溶解与媒介相符
- 复古或故障美学，粗瓷砖网格让人想起像素时代的屏幕擦除
- 方向性揭示（对角线、放射状），暗示眼睛接下来该往哪里走

## 工作原理
在离开的页面上方建一个 `n × n` 空瓷砖的网格覆盖层。每块瓷砖从一个图案函数得到各自的 `transition-delay`，然后所有瓷砖同时淡到不透明——错开的延迟产生了溶解。页面切换发生在中途，藏在瓷砖下面，覆盖层被清除（或者可选地以对称的溶解淡出，露出新页面）：

```js
function getDelays(n, style) {
  const total = n * n;
  if (style === 'random') {                       // shuffle tile order
    const order = Array.from({length: total}, (_, i) => i).sort(() => Math.random() - 0.5);
    return order.map((_, i) => order.indexOf(i) / total);
  }
  if (style === 'diagonal') {                      // (row + col) sweep
    return Array.from({length: total}, (_, i) => {
      const r = Math.floor(i / n), c = i % n;
      return (r + c) / (2 * (n - 1));
    });
  }
  // radial: tile-center distance, normalized so the corners land at 1
  return Array.from({length: total}, (_, i) => {
    const r = Math.floor(i / n) - (n - 1) / 2, c = i % n - (n - 1) / 2;
    return Math.sqrt(r * r + c * c) / (Math.SQRT2 * (n - 1) / 2);
  });
}
```

每块瓷砖的内联样式把归一化的延迟烤进一次真实的过渡：`transition: opacity <fadeDur>ms ease <delay>ms`。延迟用纯 JS 算出来、逐个元素应用，所以浏览器处理所有实际的动画——没有逐帧循环。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 图案 | 随机 | 瓷砖出现的顺序：随机看起来像颗粒，对角线从一个角扫过，从中间向外扩散成环 |
| 瓷砖大小 | 中 | 瓷砖有多大：小是 16×16 网格（手机上 10×10），中 8×8，大 4×4；小瓷砖接近平滑的淡入淡出 |
| 速度 | 普通 | 瓷砖需要多长时间：慢是 1300ms，普通 800ms，快 500ms；页面在 60% 处、在瓷砖下面换掉 |
| 双向溶解 | 关 | 也按同样的图案把瓷砖淡出，露出新页面，而不是一下子去掉它们 |

## 生产注意事项
- **随机需要一个稳定的顺序。**洗一次瓷砖顺序、把每块瓷砖映射到它的名次，能让图案保持连贯；每帧重新随机就会闪烁。这个演示洗一个顺序数组并重复使用它。
- **瓷砖数量是一个成本旋钮。**16×16 网格是 256 个动画元素。淡入淡出本身很便宜，因为每块瓷砖只过渡 `opacity`，但创建和设置这么多元素的样式会让开始溶解的那一帧在手机上变慢，所以演示在手机上把小画成 10×10。更细的网格还消耗布局和内存；超过约 24×24 后，画布或噪点纹理遮罩是更好的工具。
- **换页时机。**下面的页面在足够多的瓷砖不透明、能遮住它的时候换掉（这里是时长的约 60%）。换太早，进入的页面会透过缝隙显示；换太晚，揭示感觉迟缓。
- **等价的库。**View Transitions API 可以用带遮罩的 `::view-transition-old` 做溶解，但开箱即用不提供逐瓷砖错开——生成一张遮罩图片是原生的做法。GSAP 的 `stagger`（带 `grid` 和 `from: 'random'`）直接复现了这一点。基于着色器的溶解对一个上升的阈值采样噪点纹理，像素粒度上是同一个思路。

## 另见
- [交叉淡入淡出过渡](../crossfade/) — 没有瓷砖的均匀淡入淡出
- [闪光 / 漏光过渡](../flash-transition/) — 一道闪光藏住切换
- [模糊过渡](../blur-transition/) — 模糊藏住切换
- [滑动过渡](../slide-transition/) — 页面移动而不是溶解
- [碎裂效果](../../02-entrance-and-exit/shatter-effect/) — 一张卡片碎成飞散淡出的碎片
- [像素化过渡](../pixelate-transition/) — 页面碎成大像素，再变清晰成下一页
