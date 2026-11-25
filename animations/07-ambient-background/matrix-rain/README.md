# 矩阵雨

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/matrix-rain/)

## 它是什么
矩阵雨，也叫数字雨，是字符列在深色屏幕上落下的背景。每列的领头字符明亮，后面的字符向黑色淡出，所以每列看起来像一个发光的头带着渐隐的尾巴。电影《黑客帝国》（1999）之后，它成了“电脑在工作”的速记。

## 什么时候用
- 终端、黑客或“系统在线”主题的落地页和主视觉区
- 复古未来或赛博朋克界面的加载和启动画面
- 登录面板、404 页面或活动倒计时后面的装饰背景
- 音乐可视化和演示场景风格的氛围循环

## 工作原理
每列跟踪一个竖直位置（它的“水滴”）。每步，字符列在头部画一个新鲜明亮的字符，然后前进一步。拖尾不是存储的——它是每步在整个画布上画一个半透明黑色矩形产生的，这让之前画的每个字符再暗一点，直到消失：

```js
// Fade the whole canvas slightly instead of clearing it.
ctx.fillStyle = 'rgba(0,0,0,' + FADE + ')';   // FADE ~0.07
ctx.fillRect(0, 0, W, H);

const pitch = W / cols;
for (let i = 0; i < cols; i++) {
  const x = i * pitch, y = drops[i] * fontSize;
  ctx.fillStyle = '#d8ffe6';                    // bright leader
  ctx.fillText(pick(), x, y);
  if (y > H && Math.random() > 0.975) drops[i] = Math.floor(Math.random() * -20);  // back above the top, at random
  drops[i]++;                                   // fall one row
}
```

因为画布永远不会被完全清空，尾巴的长度完全由淡出不透明度控制：值越小，字符在屏幕上留得越久（长尾巴），值越大，擦除越快（短尾巴）。字符列越过底部后随机回到顶部，所以雨永远不会同步落下。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 雨每秒向下移动一行的次数：慢速约 18 次，普通 28 次，快速约 46 次 |
| 字符 | 日文 | 日文片假名加数字 0 到 9（电影的样子）、二进制 0 和 1，或十六进制 0 到 F |
| 轨迹长度 | 中等 | 字符留存多久：短是每步淡 12%，中等 7%，长是 4% |
| 字符大小 | 中等 | 小是 12px，中等 16px，大是 22px；字符列按一个字符大小的间距排列，手机上至少 14px，所以字符越大列越少 |
| 发光的领头字符 | 开 | 每列领头字符周围柔和的辉光 |

## 生产注意事项
- **`fillRect` 淡出与 `clearRect`**：每帧清空画布没有拖尾——你得自己存储、重画每个过去的字符。画半透明的 `fillRect` 既是拖尾机制又更便宜，因为它是一个合成的矩形，而不是几百个保留的格子。
- **手机上限制密度**：列数由最小像素间距得出，不是无上限的 `width / fontSize`。加上把 `devicePixelRatio` 限制为 2，这让每帧 `fillText` 的数量有上限，效果在中端手机上保持 60fps。重的 `shadowBlur` 是最贵的部分——它只用在单个领头字符上，绝不用在拖尾上。
- **隐藏标签页时暂停**：`document.hidden` 为真时在 `visibilitychange` 上取消 `requestAnimationFrame` 循环，标签页回来时重启，所以后台标签页不耗 CPU 或电池。
- **减少动态**：演示开始时是暂停的，显示一次画成的满屏雨，直到访客按下播放。生产环境里，给这些访客显示一帧静止画面。
- **它是装饰性的**：实际使用应该位于内容后面，带 `aria-hidden="true"`、`pointer-events: none`，前景文字对比度足够，雨不会降低可读性。把它当作质感，而不是信息。
- **不需要库**：2D 画布、`fillText` 和半透明的 `fillRect` 就是整个效果。

## 另见
- [星空](../starfield/) — 另一个移动点的画布背景
- [扫描线效果](../scanline/) — 同样老终端氛围的深色线条
- [合成波网格](../synthwave-grid/) — 复古未来背景的霓虹网格
- [ASCII / 半调](../../06-3d-advanced/ascii-halftone/) — 用字符重画的动态画面，同样复古
- [生命游戏](../game-of-life/) — 按简单规则生死的细胞网格
