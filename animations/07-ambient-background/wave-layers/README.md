# 波浪层

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/wave-layers/)

## 它是什么
波浪层把几条水带一层层叠起来，每条顶边是波浪形。每条带子沿自己的无尽循环横向滑动，近处的带子位置更低、移动更快、波浪更大，所以几个扁平形状看起来就是深水。浏览器不用脚本就让它们全部动起来。

## 什么时候用
- 主视觉区底部边缘，波浪引向下方的页面
- 旅行、海滩、航海和户外网站的页脚和分区隔线
- 健康、冥想或度假页面的平静开场
- 需要安静生命迹象的空白或等待状态

## 工作原理
每层放着一条比舞台宽两倍的轨道。轨道顶部是 SVG 波浪，其余部分由同色的纯色块填满。一条路径服务所有层：它画十个完整的波浪，每层的 `viewBox` 只显示其中前十、八、六或四个，用 `preserveAspectRatio="none"` 拉伸到整条轨道。所以远层得到小而密的波浪，近层得到长波浪。

```html
<svg class="defs"><defs><path id="wv" d="M0 0C18 0 32 40 50 40S82 0 100 0 … S982 0 1000 0V40H0Z"/></defs></svg>

<div class="wave w1"><div class="track">
  <svg viewBox="0 0 1000 40" preserveAspectRatio="none"><use href="#wv"/></svg><i></i>
</div></div>
<!-- w2, w3 and w4 show 800, 600 and 400 units of the same path -->
```

每条轨道放着偶数个波浪，所以向左滑动一半宽度后看起来和开始时完全一样，跳回起点看不出来。舞台是尺寸容器（`container-type: size`），所以高度用 `cqh` 写，是舞台高度的百分之一。每层在舞台上有自己的高度（`--y`）、波浪大小（`--k`）、周期（`--s`）和起跑提前量（`--o`）；周期和提前量都是一份 `--dur` 的份额：

```css
.track { width: 200%; animation: drift calc(var(--dur) * var(--s)) linear calc(var(--dur) * var(--s) * var(--o)) infinite; }
.track svg { width: 100%; height: calc(var(--amp) * var(--k)); fill: var(--fill); }
.wave { top: calc(var(--y) - var(--amp) * var(--k) / 2); bottom: -12cqh;
        animation: bob calc(var(--dur) * var(--s) / 4) ease-in-out calc(var(--dur) * var(--s) * var(--o)) infinite alternate; }
.w1 { --y: 42%; --k: .5; --s: 1.7; --o: -.15; }   /* far: high, small waves, slow */
.w4 { --y: 80%; --k: 1;  --s: .9;  --o: -.8;  }   /* near: low, big waves, fast */
@keyframes drift { to { transform: translateX(-50%); } }
@keyframes bob { from { transform: translateY(-1.2cqh); } to { transform: translateY(1.2cqh); } }
```

新的速度会改 `--dur`，这会让每层跳到周期的另一个点。先把每个动画的时间按相同倍数缩放，波浪就能保持在原地，动或暂停都一样：

```js
stage.getAnimations({ subtree: true }).forEach(a => { if (a.currentTime != null) a.currentTime = a.currentTime * next / secs; });
stage.style.setProperty('--dur', next + 's');
```

配色保持一条规则：天空在地平线处最亮，每个层都比那片辉光更暗，近处的最暗，所以无论显示多少层，每层都和后面的层、和天空区分开来。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 基础周期：慢速 40 秒，正常 24 秒，快速 14 秒；从远到近，各层滑动一个舞台宽度分别需要那个时间的 1.7、1.35、1.1 和 0.9 倍 |
| 颜色 | 海洋 | 海洋蓝、日落橙光下的粉紫，或夜晚靛蓝；远层最淡，近层最暗 |
| 波浪高度 | 中等 | 最近的波浪有多高：低是舞台高度的 6%，中等 11%，高 17%；更远的层只有那个数的一半到五分之四 |
| 层数 | 4 | 两、三或四层；层数较少时，先去掉远层，近处的波浪始终保留 |

## 生产注意事项
- **只有变换在动**：轨道用 `translateX` 滑动，层用 `translateY` 起伏，浏览器交给合成器处理，所以运行时什么都不重绘，手机上也是。给路径形状或 `background-position` 做动画每帧都会重绘。
- **接缝**：每条波浪下的色块从高一像素处开始（`margin-top: -1px`），所以在任何缩放级别，它和 SVG 之间都不会出现细线。
- **遮罩变体**：同样的带子也可以是带纯色背景和重复波浪 `mask-image` 的单个元素；遮罩这时设定波长（`mask-size`），不需要任何 SVG 标记。
- **减少动态**：演示让波浪停在原地。在你自己的页面里，用一条 `prefers-reduced-motion: reduce` 规则把轨道的 `animation` 设为 `none`，效果相同。
- **库**：GSAP 可以用 `gsap.to(track, { xPercent: -50, duration, ease: 'none', repeat: -1 })` 循环每条轨道，Framer Motion 用 `animate={{ x: '-50%' }}` 和 `transition={{ repeat: Infinity, ease: 'linear', duration }}`；两者都只是替换关键帧。

## 另见
- [极光](../aurora/) — 摇曳而不是滑动的柔和光带
- [视差滚动](../../01-scroll-based/parallax-scrolling/) — 同样的近快远慢规则，由滚动驱动
- [跑马灯/滚动条](../../05-text-typography/marquee-ticker/) — 同样的无缝半宽循环，用文字
