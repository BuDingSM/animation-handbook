# 呼吸光晕

[在线演示](https://animation.xizanghuapi.cn/animations/07-ambient-background/breathing-glow/)

## 它是什么
呼吸光晕是一团柔和的圆形彩色光晕，慢慢变大变亮，再缩小变暗，反反复复，像一个人平静地呼吸。四到六秒一个周期符合放松的呼吸。它位于有生气但处于静止的东西后面，比如暂停的音乐播放器、闲置的语音助手或冥想计时器。

## 什么时候用
- 语音助手、AI 聊天界面和环境计算产品的闲置或“休息”状态
- 暂停或停止的媒体播放器后面，画面应该确认“它准备好了”
- 冥想、专注或呼吸练习应用，画面引导呼吸
- 标志揭示和品牌时刻，单个元素值得拥有存在感和平静

## 工作原理
一个径向渐变元素用 CSS `@keyframes` 做动画，改变它的 `transform: scale()`：

```css
:root {
  --glow-dur: 5s;
  --glow-min: 0.6;
  --glow-max: 1.5;
  --glow-color: #58a6ff;
}

.glow {
  width: 260px; height: 260px;
  border-radius: 50%;
  background: radial-gradient(circle, var(--glow-color) 0%, transparent 70%);
  filter: blur(40px);
  opacity: 0.6;
  animation: breathe var(--glow-dur) ease-in-out infinite;
}

@keyframes breathe {
  0%, 100% {
    transform: scale(var(--glow-min));
    opacity: 0.6;
  }
  50% {
    transform: scale(var(--glow-max));
    opacity: 0.9;
  }
}
```

**双层光晕变体**——第二层光晕的周期更长、略有不同，造出有机的不同步：

```css
.glow-outer {
  width: 390px; height: 390px;
  opacity: 0.3;
  filter: blur(60px);
  animation: breathe calc(var(--glow-dur) * 1.3) ease-in-out infinite reverse;
}
```

`reverse` 方向和 1.3 倍时长确保外层光晕永远不会与内层光晕同步，让组合效果感觉更有机。

**配对元素呼吸**——中心元素也用自己的动画微微缩放：

```css
.center-el {
  animation: el-breathe var(--glow-dur) ease-in-out infinite;
}
@keyframes el-breathe {
  0%, 100% { transform: scale(1);    opacity: 0.9; }
  50%       { transform: scale(1.02); opacity: 1;   }
}
```

中心元素 2% 的缩放是特意做得微妙的——眼睛不应该有意识地注意到元素在动。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 一次呼吸需要的时间：慢速 8 秒，普通 5 秒，快速 3 秒；4 到 6 秒感觉放松，2 秒让人焦虑 |
| 变大多少 | 中等 | 光晕缩小、长大多少：一点点是它大小的 80% 到 120%，中等 60% 到 150%，很多是 45% 到 180% |
| 光晕颜色 | 蓝色 | 光晕的颜色：粉色、蓝色、紫色、绿色或橙色（白色会在呼吸顶峰处藏住白色标题） |
| 光晕大小 | 中等 | 光晕长大前的大小：小是 180px，中等 260px，大是 340px |
| 第二层光晕 | 关 | 第一层后面更大、更柔和的光晕，是它的 1.5 倍大小，亮度差不多：它在与第一层相同的范围内变亮变暗，周期是 1.3 倍长，所以两者时而同步、时而错开 |
| 图标和标题也一起呼吸 | 开 | 图标和它下面的标题每次呼吸胀大 2%，微妙到有意识地注意不到 |

## 生产注意事项
- **为什么是 4–6 秒？**这个范围符合放松的成年人呼吸频率（每分钟 10–15 次）。苹果的 Siri 光球用的大约也是这个范围。这不是巧合——这个时间会产生潜意识的平静生物反馈。
- **`filter: blur()` 与 `border-radius: 50%`**：两者结合不用图片就能造出圆形的柔和光晕。`filter: blur()` 是 GPU 加速的，在所有缩放值下都平滑。
- **缓动**：`ease-in-out` 是慢启动、到中点轻柔加速、在顶峰慢减速——符合呼吸真实的感觉（吸气开始和结束都很慢）。
- **`will-change: transform`**：给光晕元素加上它，提示浏览器分配一个合成层。多个光晕叠加时有用；单个光晕没必要。
- **Framer Motion**：`<motion.div animate={{ scale: [0.6, 1.5, 0.6] }} transition={{ duration: 5, repeat: Infinity, ease: "easeInOut" }} />`——用 React 得到相同的结果。

## 另见
- [环境涟漪](../ambient-ripple/) — 圆环扩散开来，而不是光晕胀大
- [网格渐变](../mesh-gradient/) — 柔和的色彩在整个背景上漂移
- [漂浮元素](../floating-elements/) — 漂移、慢慢淡入淡出的图形
