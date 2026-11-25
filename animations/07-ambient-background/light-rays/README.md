# 光束

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/light-rays/)

## 它是什么
光束（也叫上帝之光或太阳光）是从视野外一个明亮点扇形展开的柔和光柱，就像阳光透过窗户或树间洒落。每束光边缘柔和，在光进来的地方最亮、越远越淡。光束微微摇摆，按自己缓慢的周期变亮变暗，所以光看起来扫过场景，尘埃微粒只在光束照到的地方闪烁。

## 什么时候用
- 应该感觉平静、温暖或有点电影感的主视觉区和页面开场
- 健康、自然、旅行、教堂和活动网站，光承载着氛围
- 产品发布和揭示，产品或标题位于光中
- 需要深度、又不要繁忙图案的深色部分

## 工作原理
每束光是在画布上用锥形渐变画出的柔和楔形，中心在光源处：两边透明、中间最亮。楔形用 `lighter` 混合绘制，所以两束光重叠处光会叠加，盖在覆盖整个扇形的淡淡薄雾之上。然后一个径向渐变用 `destination-in` 绘制，随离光源的距离淡出一切。

```js
ctx.globalCompositeOperation = 'lighter';
for (const r of rays) {
  r.now = base + r.c + r.s * Math.sin(r.f * t + r.p) + sweep;   // the beam sways on its own cycle
  r.al = r.a * (0.6 + 0.4 * Math.sin(r.g * t + r.q));          // and brightens and dims on another
  const g = ctx.createConicGradient(r.now - r.w / 2, sx, sy), k = r.w / (2 * Math.PI);
  g.addColorStop(0, `rgba(${rgb},0)`);
  g.addColorStop(k * 0.2, `rgba(${rgb},${r.al * 0.3})`);
  g.addColorStop(k * 0.5, `rgba(${rgb},${r.al})`);             // brightest along its middle line
  g.addColorStop(k * 0.8, `rgba(${rgb},${r.al * 0.3})`);
  g.addColorStop(k, `rgba(${rgb},0)`);
  ctx.fillStyle = g;
  ctx.beginPath(); ctx.moveTo(sx, sy);                          // a triangle a little wider than the wedge
  ctx.lineTo(sx + Math.cos(r.now - r.w) * R, sy + Math.sin(r.now - r.w) * R);
  ctx.lineTo(sx + Math.cos(r.now + r.w) * R, sy + Math.sin(r.now + r.w) * R);
  ctx.fill();
}
ctx.globalCompositeOperation = 'destination-in';                // fade with distance from the source
ctx.fillStyle = fade; ctx.fillRect(0, 0, W, H);
```

每粒尘埃带着轻柔的摇摆向上、向侧面漂移。它的亮度就是它所在位置的光：每束盖住它的光都加上自己的强度，沿光束中线最强，再乘以同样的距离淡出，所以微粒在光束经过时闪烁、在黑暗中几乎看不见。光有自己的时钟，按自上一帧以来的时间乘以速度增长（慢动作里是三分之一），一次最多 50ms，所以光束在每个屏幕上以相同节奏摇摆；每帧清空画布、重画整幅画面。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 光的颜色 | 暖色 | 棕黑色房间上的暖色傍晚阳光，或蓝黑色房间上的冷色月光 |
| 速度 | 普通 | 光束摇摆和尘埃漂移的速度：慢速是正常速度的一半，快速是两倍；正常时一束光每 12 到 22 秒来回摆动一次，一粒尘埃每秒上升 2 到 7px |
| 漂浮的尘埃 | 开 | 尘埃微粒慢慢向上漂移，在光束经过处闪烁；手机上显示 40 粒，其他屏幕 80 粒 |
| 光束数量 | 一些 | 少是 5 束，一些是 8 束，多是 12 束；更多光束重叠成更饱满、更亮的光 |
| 光从哪里来 | 左上 | 光束从哪里开始：左上角正上方、顶部边缘中间或右上角 |

## 生产注意事项
- **绘制便宜**：一帧是一次薄雾填充、每束光一个渐变楔形、一次淡出、一次辉光和几十个圆点，在笔记本上不到一毫秒。什么都不模糊：柔和边缘来自渐变本身，比模糊滤镜便宜得多。
- **手机**：手机尺寸的屏幕（宽最多 600px，或横持手机时高最多 500px）上漂浮的尘埃减半。
- **浏览器支持**：画布上的 `createConicGradient` 在所有当前浏览器里都有（Chrome 99、Safari 16.1、Firefox 112）。演示先检查它；没有它的地方，每束光是更窄的单色淡淡楔形，薄雾省略，所以页面仍然显示光而不是失败。
- **CSS 替代方案**：光束也可以是长而窄的元素，上面有柔和的渐变，用 `transform: rotate()` 绕光源转动，用遮罩淡出。保持数量少、窄，因为每个都是内存里的大图层，尤其在高密度手机屏幕上。
- **WebGL**：游戏和 3D 场景用图像明亮部分的径向模糊（体积光散射）做上帝之光；three.js 在后期处理库里有 `GodRaysEffect`。
- **减少动态**：演示开始时是暂停的，光束画成静止的，直到访客按下播放。
- **保持文字可读**：光束向底部淡出，标题就位于底部，柔和的深色阴影（文字后面的径向渐变）让它在几束光交叉的下方仍然可读；小文字远离光最亮的部分。

## 另见
- [漏光](../light-leak/) — 暖光不定时地从角落漫入
- [极光](../aurora/) — 彩色光带在夜空中摇曳
- [呼吸光晕](../breathing-glow/) — 一个柔和的光晕慢慢变大变小
