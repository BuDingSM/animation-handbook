# 微光扫过效果

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/shimmer-effect/)

## 它是什么
微光是一条光带，在内容加载时一次又一次扫过灰色占位形状。脉冲让每个色块一起上下淡出，微光朝一个方向移动，所以占位符看起来像内容在流进来。

## 什么时候用
- 内容从服务器流进来的卡片和列表骨架加载器
- 高端或有品牌的加载体验，单靠骨架脉冲感觉太平淡
- 任何骨架组件，运动方向和数据源匹配（左对齐的内容从左到右）

## 工作原理
一个装着半透明渐变的 `::after` 伪元素盖住每个骨架块，它的背景位置从一边滑到另一边：

```css
:root {
  --shim-dur: 1500ms;
  --shim-bright: 0.25;
}

.skel-card {
  position: relative;
  overflow: hidden;
}

.skel-card::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    90deg,
    transparent 25%,
    rgba(255,255,255,var(--shim-bright)) 50%,
    transparent 75%
  );
  background-size: 200% 100%;
  animation: shimmer var(--shim-dur) linear infinite;
}

@keyframes shimmer {
  from { background-position: 200% 0; }
  to   { background-position: -200% 0; }
}
```

`background-size: 200% 100%` 和位置动画，比单独用 `translateX` 对高亮宽度有更多控制。贴片是块的两倍宽，它的位置每个周期走过四个块宽，所以每个 `--shim-dur` 里光带会穿过两次：默认 1.5 秒时每 0.75 秒一次。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 光多长时间穿过一次：慢是每 1.2 秒，普通每 0.75 秒，快是每 0.45 秒；比大约每 0.4 秒更频繁会感觉慌乱 |
| 亮度 | 中等 | 光带有多强：柔和、中等和明亮在中心分别是 12%、25% 和 45% 的透明度；轻柔很微妙，明亮适合有品牌的屏幕 |
| 高亮角度 | 竖直 | 竖直扫过一条直带；倾斜把它倾斜 25°，对角 45°；整页保持一个角度 |
| 高亮颜色 | 白色 | 白色在任何占位符上都适用；蓝色或金色适合有品牌的屏幕 |

## 生产注意事项
- **只用一个方向**：多个微光组件同时朝不同方向扫，会造成视觉混乱。一个页面上的所有骨架都应该朝同一个方向闪光。
- **需要 `overflow: hidden`**：微光伪元素超出卡片。没有溢出裁剪，它会渗进相邻元素。
- **和脉冲组合**：二选一——脉冲和微光一起是多余的，视觉上很吵。
- **性能**：微光用 `background-position` 动画，而不是 `transform`。虽然 `transform` 通常更受青睐，但 GPU 合成图层上的 background-position 性能可以接受。或者，在伪元素上用 `translateX` 加 `will-change: transform`，是性能最好的做法。
- **纯 CSS**：不需要 JavaScript。动画是无限的，元素从 DOM 里移除时自动停止。
- **React**：`react-loading-skeleton` 包含微光。自定义组件时，上面的 CSS 模式和框架无关。

## 另见
- [骨架屏](../skeleton-loader/) — 占位符改为脉冲
- [加载旋转指示](../loading-spinner/) — 用于不知道多长等待的旋转指示
- [进度动画](../progress-animation/) — 显示完成了多少的横条
- [渐变边框动画](../gradient-border/) — 一条色带绕着边缘跑，而不是横扫
