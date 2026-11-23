# 通知徽标脉冲

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/badge-pulse/)

## 它是什么
通知徽标是图标上一个小的彩色圆点或数字，标记有新内容，比如未读消息。轻柔的脉冲——长大一点再收回，或向外送出一圈柔和的光环——能在余光里抓住眼球，又不会打断你正在做的事。

## 什么时候用
- 有未读消息数的通知铃铛
- 表示有新消息的收件箱或聊天图标
- 表示在线状态的头像状态圆点
- 任何必须一直可见、又不能打扰人的持久指示

## 工作原理
徽标通过循环的 `scale` 关键帧动画来脉冲：

```css
:root {
  --pulse-dur: 1500ms;
  --pulse-scale: 1.3;
  --badge-color: #f85149;
}

.badge {
  position: absolute;
  top: -8px; right: -8px;
  min-width: 22px; height: 22px;
  border-radius: 11px;
  background: var(--badge-color);
  border: 2px solid var(--page-bg);
  animation: badge-scale var(--pulse-dur) ease-in-out infinite;
}

@keyframes badge-scale {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(var(--pulse-scale)); }
}
```

光晕变体再加一圈扩散淡出的环。环画在徽标数字的下面，而不是上面：徽标是自己的层叠上下文（`isolation: isolate`），环的 `z-index` 是负数，所以它盖住徽标的颜色，但永远盖不住数字，这样每次脉冲数字的对比度都保持不变：

```css
.badge {
  isolation: isolate;
}

.badge::after {
  content: '';
  position: absolute;
  inset: -4px;
  z-index: -1;
  pointer-events: none;
  border-radius: 50%;
  background: var(--badge-color);
  opacity: 0.4;
  animation: halo-grow var(--pulse-dur) ease-out infinite;
}

@keyframes halo-grow {
  0%   { transform: scale(1); opacity: 0.5; }
  100% { transform: scale(2.2); opacity: 0; }
}
```

环长到徽标的 2.2 倍，远远超出图标的角落，而且即使淡出后它也能接到指针。所以它有 `pointer-events: none`；没有它，点图标旁边空白处时会点到环上，算成点到了徽标。

三次脉冲后停止，需要在动画部分再加两个声明：`animation-iteration-count: 3` 和 `animation-fill-mode: forwards`。没有填充模式，最后一次脉冲结束时光晕会回到静止样式——徽标周围一圈淡淡的环；有了它，这次运行会停在最后一帧关键帧上，徽标是 scale 1，光晕看不见。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 脉冲样式 | 变大 | 变大把徽标放大再收回；圆环从徽标周围送出一圈柔和的光晕，长到 2.2 倍大小，同时从一半不透明淡出到完全透明；两者都把两种做法合在一起 |
| 速度 | 普通 | 一次脉冲要花多长时间：慢是 2.4s，普通 1.5s，快 0.9s；快显得紧急，慢容易被忽略 |
| 变大多少 | 中等 | 每次脉冲顶部徽标变得有多大：稍微是 115%，中等 130%，很多是 150% |
| 三次后停止 | 关 | 徽标脉冲三次然后停下；在线圆点会一直脉冲 |
| 徽标颜色 | 红色 | 徽标的颜色；红色表示有新内容、很紧急 |

## 生产注意事项
- **余光感知阈值**：1.5s 内 1.15 倍的缩放，是大多数人在余光里能察觉的最小值。低于它，徽标看起来就是静止的。
- **交互后消除**：用户查看通知后，一定要移除脉冲（以及徽标本身）。已经看过的内容上还留着脉冲，会让人困惑。
- **引起注意后淡出**：前 5 秒强烈脉冲，然后减小缩放或完全停止。这和真实通知系统的行为一致（提醒已经在余光里“被看见”了）。
- **用 `border: 2px solid` 背景色的技巧**：这让徽标看起来浮在图标表面上，中间有一道缝。如果图标放在不均匀的背景上，记得更新边框颜色。
- **`prefers-reduced-motion`**：减弱动态效果下，演示开始时是暂停的，所以徽标保持静止，直到访客按下「播放」。生产环境里请关掉脉冲：徽标保持可见，只是不动。
- **React**：`react-hot-toast` 和 Sonner 为轻提示实现了徽标风格的指示。对于导航徽标，大多数组件库都有带可选 `animate` 属性的 `Badge`。

## 另见
- [提示气泡浮现](../tooltip-reveal/) — 指向图标会显示一条简短说明
- [加载旋转指示](../loading-spinner/) — 另一个表示正在发生什么的标志
- [悬停状态动画](../hover-state/) — 指针移到上面时项目会有反应
- [加入购物车飞入](../add-to-cart-fly/) — 商品飞入时购物车的数量加一
