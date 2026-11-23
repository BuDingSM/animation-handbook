# 旋转进入

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/rotate-in/)

## 它是什么

旋转进入让元素进入时绕着自己的中心旋转，通常同时从小放大，所以读起来像是在到达，而不只是在原地转。形状在每个角度看起来都对的时候，旋转最容易跟上，所以它适合圆形或对称的标记，比如图标、星星、齿轮和徽章，不适合文字或宽矩形。

## 什么时候用
- 图标和徽章揭示——成就解锁、状态指示、加载到完成的切换
- Logo 或徽章，旋转强化了「汇合」的那一刻
- 能承受一次精力充沛入场的小装饰元素
- 任何静止形状足够对称、旋转中途的一帧看起来仍然是有意的地方

## 工作原理
元素从旋转、缩小开始，加上 `.in` 类时两者都过渡回静止值。弹回缓动让旋转稍稍超过一点，感觉像落定了一样：

```css
.icon{
  opacity:0;
  transform:rotate(var(--rot-start)) scale(var(--scale-start));
  transition:transform var(--dur) var(--ease),
             opacity var(--opacity-dur) var(--ease)}
.icon.in{opacity:1;transform:rotate(0deg) scale(1)}
```

每个设置只改一个自定义属性。起始角度就是选好的旋转量，由方向决定正负，关掉弹跳会把弹回曲线换成平滑曲线：

```js
function apply(){
  vars.setProperty('--rot-start',dir*turn+'deg');
  vars.setProperty('--dur',dur()+'ms');
  vars.setProperty('--ease',bounceTog.checked?SPRINGY:SMOOTH);
  vars.setProperty('--scale-start',scaleTog.checked?'0':'1');
  vars.setProperty('--opacity-dur',fadeTog.checked?'var(--dur)':'0s');
}
```

负的起始角度顺时针转到位，正的逆时针转到位；不管怎样，元素都会落在 `rotate(0deg)`。每次播放都先在关掉过渡的情况下把图标一下子放回起始姿势，所以重播总是从头开始转进来。慢动作把时长乘以三。

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 旋转幅度 | 半圈 | 进来时转多远：四分之一圈、半圈或一整圈 |
| 速度 | 普通 | 旋转要多长时间：「慢」是 1000ms，「普通」600ms，「快」350ms |
| 旋转方向 | 顺时针 | 落下时往哪个方向转 |
| 结尾弹一下 | 开 | 稍稍超过静止角度再退回来，这样旋转会落定，而不是滑着停下 |
| 从小放大 | 开 | 从很小开始，旋转时放大，这样旋转就变成了一次到达 |
| 淡入 | 开 | 旋转时淡入，而不是一下子出现 |

## 生产注意事项
- **对称很重要**：旋转文字或矩形看起来像倒下了，而不是进来了。演示故意用了一个对称的图标，这样每一帧中间画面读起来都是对的。
- **缩放卖的是到达感**：只有旋转是让元素在原地转；加上 `scale(0) → scale(1)` 会让它感觉像是从无到有飞进来的。两者结合才产生了「落定」的感觉。
- **把不透明度的时间分开**：演示给淡入淡出自己的时长（`--opacity-dur`），这样比如可以在旋转继续的时候快速淡入——否则完全透明的旋转开始可能看起来像故障。
- **减弱动态**：在 `prefers-reduced-motion` 下，图标退到单纯的 300ms 不透明度淡入淡出，没有旋转或缩放，演示也不循环。
- **Framer Motion**：在变体里一起给 `rotate` 和 `scale` 做动画，用弹簧过渡来做落定的超过。
- **GSAP**：用 `back.out` 缓动给 `rotation` 和 `scale` 做补间；`back` 用一个关键词就提供了超过。

## 另见
- [翻转进入](../flip-in/) — 像卡片翻面一样摆进来
- [缩放进入](../scale-in/) — 从小放大，没有旋转
- [弹跳进入](../bounce-in/) — 带着弹簧般的弹跳落定
