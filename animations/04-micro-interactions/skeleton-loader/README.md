# 骨架屏

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/skeleton-loader/)

## 它是什么
骨架屏按即将到来的内容的布局，显示灰色占位形状。色块轻轻脉冲，表示正在加载；真实内容到来时，在它们上面淡入。因为人们看到了即将到来的东西的形状，等待比面对空白区域或旋转指示时感觉更短。

## 什么时候用
- 内容从 API 到来的社交信息流、卡片和列表
- 独立加载的仪表盘小部件
- 图片加载前尺寸已知的图片画廊
- 任何用户本来会盯着一片空白的界面

## 工作原理
骨架块是普通的 `<div>` 元素，按预期内容的形状做了样式。一段 CSS 动画让它们的透明度脉冲：

```css
.skel {
  background: #1a1f28;
  border-radius: 4px;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.5; }
  50%       { opacity: 1;   }
}
```

真实内容到来时，把骨架淡出，内容淡入：

```js
async function load() {
  showSkeleton();
  const data = await fetchData();
  hideSkeleton();
  showContent(data);
}

function hideSkeleton() {
  skeletonEl.style.display = 'none';
  contentEl.style.opacity = '1'; // triggers CSS transition
}
```

内容元素需要 `opacity: 0; transition: opacity 400ms ease` 才能平滑淡入。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 脉冲速度 | 普通 | 一次脉冲要花多长时间：慢是 2.4 秒，普通 1.5 秒，快是 0.9 秒；一到两秒感觉平静，更快会感觉焦虑 |
| 脉冲强度 | 中等 | 色块在两次脉冲之间淡出多远：柔和、中等和强烈分别把它们淡到 70%、50% 和 30% 的透明度，再回到全亮；轻柔的脉冲就够了 |
| 加载时间 | 中等 | 占位符在内容淡入之前显示多长时间：短是 1 秒，中等 2 秒，长是 3.5 秒 |

## 生产注意事项
- **形状匹配很重要**：骨架和到来的内容不匹配，会造成布局偏移，破坏效果。量出真实内容的尺寸并匹配它们。
- **不要同时动画太多骨架**，一个页面上——3–4 个独立脉冲的元素是上限，超过屏幕会开始感觉混乱。
- **真实数据的近似**：如果知道内容长度，按它动态调整骨架线的大小（比如人名通常 1–2 行；描述 3–4 行）。
- **`prefers-reduced-motion`**：为要求它的用户完全关掉脉冲动画。静态的灰色块仍然是有效的骨架加载器。
- **React**：`react-loading-skeleton`（Dvtng 出品）是标准库。它自动把骨架宽度匹配到行内文本节点。
- **脉冲还是微光**：骨架要么像这里一样脉冲显示加载，要么用扫过的光带（[微光扫过效果](../shimmer-effect/)）；二选一，两者一起是多余的，视觉上很吵。

## 另见
- [微光扫过效果](../shimmer-effect/) — 一条光带扫过占位符
- [加载旋转指示](../loading-spinner/) — 用于不知道多长等待的旋转指示
- [进度动画](../progress-animation/) — 显示完成了多少的横条
