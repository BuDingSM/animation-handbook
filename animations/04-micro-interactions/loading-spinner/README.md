# 加载旋转指示

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/loading-spinner/)

## 它是什么
加载旋转指示是一个一直在循环里动的小图形，表示系统正在忙。它不说要等多久，只说工作正在进行，所以适合不知道多长的等待，比如网络请求。演示展示了六种常见设计。

## 什么时候用
- 不知道响应时间的网络请求
- 身份验证流程、支付处理和其他服务器往返操作
- 用户触发但无法取消的后台操作
- 点击后按钮内的加载状态（短暂地替换标签）

## 工作原理
经典的圆环旋转指示用 `border` 技巧：一个完整的圆，一个象限涂成不同的颜色，不停旋转：

```css
:root { --spd: 800ms; --clr: #58a6ff; --sz: 40px; }

.sp-ring {
  width: var(--sz); height: var(--sz);
  border-radius: 50%;
  border: 3px solid rgba(255,255,255,.1);
  border-top-color: var(--clr);
  animation: spin var(--spd) linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }
```

SVG 弧线变体对弧长有更多控制，弧长也可以做动画：

```css
.sp-arc circle {
  fill: none;
  stroke: var(--clr);
  stroke-width: 4;
  stroke-linecap: round;
  stroke-dasharray: 60 100;
  animation: arc-pulse calc(var(--spd)*2) ease-in-out infinite;
}

@keyframes arc-pulse {
  0%,100% { stroke-dasharray: 15 100; }
  50%     { stroke-dasharray: 80 100; }
}
```

弹跳圆点变体在三个同级元素上用交错的动画延迟：

```css
.dot { animation: bounce var(--spd) ease-in-out infinite; }
.dot:nth-child(2) { animation-delay: calc(var(--spd) * 0.15); }
.dot:nth-child(3) { animation-delay: calc(var(--spd) * 0.30); }

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-14px); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 转一圈要花多长时间：慢是 1.3 秒，普通 0.8 秒，快是 0.5 秒；0.6 到 1 秒读起来平静，更快会显得焦虑 |
| 大小 | 中等 | 小是 28px，中等 40px，大是 52px；小号放得进按钮，大号填满空白区域 |
| 颜色 | 蓝色 | 旋转指示的颜色；选一个和页面对比明显的颜色 |

## 生产注意事项
- **最短显示时间**：如果操作在大约 400ms 内完成，要么完全不显示旋转指示，要么强制一个最短显示时间。短暂闪现的旋转指示，造成的困惑比它解决的更多。
- **按钮内的旋转指示**：点击时用旋转指示替换按钮标签，收到响应时恢复。这个模式可以防止重复提交。
- **`role="status"` 和 `aria-label`**：读屏软件需要播报加载状态。给旋转指示的容器加上 `role="status"` 和 `aria-label="Loading"`。
- **`prefers-reduced-motion`**：减弱动态效果下演示一开始是暂停的，所以旋转指示保持静止，直到访客按“播放”。生产中把旋转指示简化成简单的透明度脉冲，或隐藏它，依靠 `aria-live` 播报。
- **React 生态**：`react-spinners`（David Hu 出品）有 15 种以上的变体。Tailwind 里，在带边框的圆形 div 上用 `animate-spin` 工具类。
- **什么时候不用**：如果知道总进度（文件上传、多步流程），改用进度条——它传达的信息更多，也更让人安心。

## 另见
- [进度动画](../progress-animation/) — 显示完成了多少的横条
- [骨架屏](../skeleton-loader/) — 代替内容的灰色形状
- [对勾绘制](../checkmark-draw/) — 等待结束后的成功标志
- [正在输入指示](../typing-indicator/) — 聊天框里的三个圆点表示有人在输入
