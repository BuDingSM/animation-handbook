# 霓虹闪烁

[在线演示](https://animation.xizanghuapi.cn/animations/05-text-typography/neon-flicker/)

## 它是什么
霓虹闪烁让一个词看起来像发光的霓虹招牌，灯光有时会断断续续。字母大多数时间稳定地亮着，然后在短暂而不均匀的爆发里熄灭又回来，就像磨损的灯管或松动的电线那样，有一个字母还会像快要坏掉的灯管一样自己嗡嗡作响。

## 什么时候用
- 酒吧、餐馆、俱乐部和深夜商店的招牌和 Logo
- 夜晚、城市和复古主题，黑暗墙上的招牌定下氛围
- 音乐、游戏和活动页面，想要标题里有一点生气
- “正在营业”或“正在直播”的徽章，应该时不时吸引眼球

## 工作原理
招牌是同一组字母的两个副本，叠在一个网格格子里。下面的副本是未点亮的玻璃，用这个颜色暗淡的色调。上面的副本是光：一层浅色核心，辉光由几层 `text-shadow` 组成，墙上的颜色晕由它的 `::before` 画出来。辉光只画一次，永远不变；闪烁只给上面副本的 `opacity` 做动画，这样浏览器可以混合已经画好的层，而不是每一帧都重新画辉光：

```html
<div class="sign" role="img" aria-label="OPEN">
  <div class="layer glass" aria-hidden="true"><span class="word">…</span></div>
  <div class="layer lit" aria-hidden="true"><span class="word">…</span></div>
</div>
```

```css
.layer{grid-area:1/1}
.lit{
  color:color-mix(in srgb,var(--neon) 30%,#fff);
  text-shadow:0 0 .02em #fff,0 0 .08em var(--neon),0 0 .2em var(--neon),0 0 .42em var(--neon);
  will-change:opacity;
  animation:flick-normal 7s step-end infinite;
}
@keyframes flick-normal{
  0%{opacity:1} 1%{opacity:.25} 2%{opacity:1} 3%{opacity:.08} 4.5%{opacity:1}
  33%{opacity:.3} 34%{opacity:1}
  57%{opacity:.1} 58.5%{opacity:.85} 59.5%{opacity:.15} 61%{opacity:1}
  80%{opacity:.4} 81%{opacity:1}
}
```

`step-end` 让每个值保持到下一个关键帧，所以灯光是啪地熄灭又亮起，而不是渐渐变化，爆发落在循环里不均匀的点上，节奏感觉不规律。每个字母在两个副本里都是自己的 span；破损字母在 4.3 秒的周期里跑第二套关键帧 `buzz`。因为 4.3 秒和 7 秒互相除不尽，字母和招牌彼此漂移，图案要很久才重复一次。闪烁频率换入另一套关键帧，它以一次短暂的闪烁开始，所以变化马上就能看到。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 颜色 | 粉色 | 灯管、辉光和墙上颜色晕的颜色 |
| 闪烁频率 | 普通 | 平静：9 秒的循环里有两次短暂爆发。普通：7 秒里有四次爆发。频繁：6 秒里有六次爆发，其中一次是大约 0.4 秒的较长熄灭 |
| 破损字母 | 开 | 一个字母在词的中部稍过一点的位置，按自己的周期嗡嗡作响并熄灭 |
| 辉光大小 | 中 | 光扩散多远：小、中或大的辉光层组合，最宽的模糊在小是字体大小的 0.12 倍，中是 0.42 倍，大是 1.1 倍 |
| 文字外框 | 开 | 加一个圆角灯管边框，和字母一起发光、闪烁 |
| 你的文字 | OPEN | 招牌上的词，最多 10 个字母，两端的空格会被去掉；它会缩小以适应舞台，清空输入框会回到 OPEN |

## 生产注意事项
- **闪烁安全**：闪烁就是闪光。让整个招牌在任何一秒里低于三次闪烁（演示最忙的设置也保持在这个范围内），大块明亮区域不要放进去；一个嗡嗡作响的字母够小，可以闪得更快一点。永远不要让全屏背景闪烁。
- **永远不要给阴影做动画**：大的 `text-shadow` 或 `box-shadow` 绘制起来很贵，给它的模糊或颜色做动画会在每一帧重绘。辉光画一次，闪烁层的 `opacity`，用 `will-change: opacity` 让它拿到自己的层。
- **减弱动态效果**：演示在稳定的辉光上开始，是暂停的，按播放只会让招牌柔和地变暗，没有闪烁。生产中，给这些访客展示稳定的辉光。
- **无障碍**：招牌对屏幕阅读器来说是一张图片（`role="img"` 加词的 `aria-label`），两个字母副本都是 `aria-hidden`，所以词只读一次。点亮的字母接近白色，在黑暗的墙上仍然可读。
- **库的等价做法**：用 GSAP 时，一条由短 `opacity` 补间组成的时间线，配合 `repeatRefresh: true` 和 `gsap.utils.random()` 的延迟，让每次爆发都不一样。Web Animations API 用 `element.animate()` 和随机的 `delay` 也能做到同样的事；Framer Motion 的 `animate` 配合 `times` 数组，贴合这里的关键帧做法。

## 另见
- [故障文字](../glitch-text/) — 文字像信号故障一样撕成红青色条带
- [文字渐变动画](../text-gradient-animation/) — 颜色在静止的文字中流动
- [呼吸光晕](../../07-ambient-background/breathing-glow/) — 柔和的辉光慢慢变大变小
- [漏光](../../07-ambient-background/light-leak/) — 暖光在随机的时间里漫进来
