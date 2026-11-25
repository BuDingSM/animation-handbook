# 漏光

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/light-leak/)

## 它是什么
漏光是一团暖光从画框边缘漫入、又淡出，像老相机密封不好时到达胶片的杂散光。真实的漏光随机发生、强度各不相同，所以效果在每次漏光之间等待一个随机间隔，每次峰值亮度也略有不同；定时均匀的漏光看起来像机器，不像胶片相机。

## 什么时候用
- 编辑和摄影作品集网站，模拟美学是特意的
- 生活方式品牌页面，“胶片摄影的暖意”是品牌标识的一部分
- 音乐视频页面和创意机构网站
- “复古胶片”的感觉能增强品牌叙事的任何场景

## 工作原理
漏光是一个绝对定位的叠加层，带一个从角落指向的径向渐变。CSS `transition` 给 `opacity` 做动画，从 0 到峰值，再回到 0。**JavaScript 随机化时间**——这是区分逼真与机械的关键细节：

```js
const leak = document.querySelector('.leak');

function flash() {
  leak.style.setProperty('--leak-dur', fadeMs + 'ms');   // the fade time, read by the CSS transition
  // each leak peaks a little below the chosen brightness, at random
  leak.style.setProperty('--leak-peak', (brightness * (0.75 + Math.random() * 0.25)).toFixed(3));
  leak.classList.add('active');                // fades in over --leak-dur
  setTimeout(fadeOut, fadeMs + 200);           // hold the peak for 200ms
}

function fadeOut() {
  leak.classList.remove('active');             // fades out over --leak-dur
  setTimeout(flash, fadeMs + randomGap());     // a random dark gap before the next leak
}

function randomGap() {
  return (minGap + Math.random() * (maxGap - minGap)) * 1000;   // 1–3s in the demo
}

setTimeout(flash, randomGap());
```

演示以一次处于峰值的漏光开场，而不是黑暗的舞台：它点亮光晕，立刻完成淡入（`leak.getAnimations().forEach(a => a.finish())`），并用 `setTimeout(fadeOut, randomGap())` 启动链条，所以第一次漏光保持一个间隔才淡出。

**左上角漏光的渐变**：

```css
.leak {
  position: absolute; inset: 0;
  pointer-events: none;
  opacity: 0;
  transition: opacity var(--leak-dur) ease-in-out;
  background: radial-gradient(
    ellipse at -10% -10%,
    rgba(255, 165, 50, 0.9)  0%,
    rgba(255, 165, 50, 0.5) 20%,
    rgba(255, 165, 50, 0.1) 50%,
    transparent              70%
  );
}

.leak.active { opacity: var(--leak-peak); }
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 光从哪里来 | 左上 | 光晕从哪个角落漫入，或中间竖向的一条光带 |
| 漏光间隔 | 中等 | 一次漏光与下一次之间的随机黑暗间隔：短是 0.5–1.5 秒，中等 1–3 秒，长是 3–8 秒 |
| 速度 | 普通 | 每次漏光淡入、再淡出需要的时间：慢速 2 秒，普通 1.2 秒，快速 0.7 秒 |
| 亮度 | 中等 | 光晕有多强：暗是 30%，中等 55%，明亮是 80%；每次漏光的峰值是这个值的随机 75% 到 100% |
| 颜色 | 琥珀色 | 琥珀色是经典漏光；金色、玫瑰色和品红也是暖色，青色看起来像更冷的胶片 |
| 漂浮的光斑 | 关 | 三个柔和的光斑从底部向上漂移，随每次漏光淡出 |

## 生产注意事项
- **不规则就是全部效果**：每 8 秒以 0.5 不透明度触发一次的漏光读起来像 CSS 动画循环，不像胶片缺陷。每次出现都随机化延迟和强度，才造出模拟的感觉。
- **颜色很重要**：金色/琥珀色是经典的漏光颜色。青色暗示蓝光漏光（更冷的胶片）。玫瑰色和品红也存在。纯白读起来像镜头光晕，不是漏光。
- **同时多次漏光**：真实相机有时有多个漏光源。加两三个带独立随机计时器的漏光元素会增加逼真感。
- **暂停**：页面被放到后台时（`document.visibilitychange`），取消待处理的 `setTimeout`。标签页不可见时触发漏光是浪费——用户回来时还可能造成刺眼的闪光。
- **Framer Motion**：每次闪光用 `<motion.div animate={{ opacity: [0, 0.5, 0] }} transition={{ times: [0, 0.5, 1], duration: 1.5 }}`。用 JavaScript 的 `controls.start()` 命令式触发。

## 另见
- [颗粒噪点叠加](../grain-overlay/) — 与漏光搭配的胶片颗粒
- [扫描线效果](../scanline/) — 页面上的深色线条，像老显示器
- [动态毛玻璃](../../06-3d-advanced/glassmorphism-animated/) — 移动色彩上的磨砂玻璃
- [光束](../light-rays/) — 从上方洒落、缓缓扫过的柔和光束
