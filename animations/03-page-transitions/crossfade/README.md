# 交叉淡入淡出过渡

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/crossfade/)

## 它是什么
交叉淡入淡出让旧页面淡出的同时新页面淡入，这样有一瞬间两个页面都半透明、彼此混合。正是这种重叠让它成为交叉淡入淡出：先让旧页面淡出、再让新页面淡入，中间会留下一段短暂的空白。演示两种方式都能做，方便你比较。

## 什么时候用
- 中性的、无方向的导航，不暗示页面之间有空间关系
- 持久外壳里的内容切换（标签页、筛选、图片画廊）
- 安全的默认过渡——当你找不到用滑动或缩放的理由时，它总能用
- 需要最平静的动效的情况，因为什么都不动，只有不透明度在变

## 工作原理
用「同时」时，两个页面得到同样的不透明度过渡，同时淡入淡出；用「一个接一个」时，每个页面得到一半的时长，新页面在旧页面消失后才开始。两者都是普通的 CSS `opacity` 过渡：

```js
if(mode==='true'){
  // Together: both fade over the full duration
  oldEl.style.transition=`opacity ${dur}ms ${ease}`;
  newEl.style.transition=`opacity ${dur}ms ${ease}`;
  oldEl.style.opacity='0'; newEl.style.opacity='1';
}else{
  // One after the other: fade old out over half, then new in over half
  const half=Math.round(dur/2);
  oldEl.style.transition=`opacity ${half}ms ${ease}`;
  oldEl.style.opacity='0';
  setTimeout(()=>{
    newEl.style.transition=`opacity ${half}ms ${ease}`;
    newEl.style.opacity='0'; newEl.classList.add('active');
    void newEl.offsetWidth;                   // commit the transparent start
    newEl.style.opacity='1';
  }, half+50);
}
```

两个页面都绝对定位在同一个层叠上下文里（`inset:0`），所以它们完美重叠，只有不透明度在变。用「同时」时，离开的页面恰好在进入的页面也到达 ~0.5 的那一刻到达 ~0.5，产生短暂的混合；用「一个接一个」时，舞台在交接处会经过一个完全空白的帧。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 淡入淡出顺序 | 同时 | 同时让两个淡入淡出重叠，页面混合在一起；一个接一个先让旧页面淡出，中间留下一段空白 |
| 速度 | 普通 | 整个切换需要多长时间：慢是 800ms，普通 500ms，快 300ms；一个接一个时，每个淡入淡出各占一半 |
| 感觉 | 轻柔 | 轻柔先慢后快再慢；平滑减速到停；均匀保持一个稳定的速度，对纯淡入淡出来说读起来最均匀 |

## 生产注意事项
- **两个页面都绝对定位**，让它们占据同一个框。如果它们让文档重排，只靠不透明度得不到干净的混合。
- **「同时」在浅色背景上会显得发白**，因为两个 50% 不透明度的图层叠在明亮的背景上，比任何一个页面单独加起来都亮。「一个接一个」以一段空白为代价避免了这个问题。
- **每次 `setTimeout` 之后加 `+50ms` 的保护**，确保过渡完成后再重置样式；没有这点余量，清理工作可能会裁掉淡入淡出的尾巴。
- **减弱动态效果**直接切换 `.active` 类，完全不用不透明度过渡。
- **等价的库**：这是 View Transitions API 默认的 `::view-transition-old/new(root)` 行为。Framer Motion 的 `AnimatePresence`（用 `initial/animate/exit` 的不透明度）、GSAP 时间线和 React Transition Group 都表达了同样的重叠。

## 另见
- [View Transitions API](../view-transitions-api/) — 浏览器自带的页面间淡入淡出
- [溶解过渡](../dissolve/) — 淡入淡出被打碎成瓷砖
- [滑动过渡](../slide-transition/) — 页面横向移动而不是淡入淡出
- [模糊过渡](../blur-transition/) — 模糊加入淡入淡出
