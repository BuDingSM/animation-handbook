# 滑动过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/slide-transition/)

## 它是什么
滑动过渡把旧页面从一侧滑出，同时新页面从另一侧滑入，像轨道上的面板一样。当方向跟着导航走——前进向左滑、后退向右滑——它会给访客一种位置感，就像手机应用那样。演示也可以总是朝同一个方向滑，或向下滑动。

## 什么时候用
- 多步骤流程：新手引导、结账、向导，「下一步」和「返回」有明确的方向
- 移动风格的导航栈，推入和弹出屏幕时应该有空间感
- 轮播和分页内容，有内在的从左到右顺序
- 任何运动方向应该强化导航方向的地方

## 工作原理
方向由目标索引大于还是小于当前索引得出；前进返回 `-1`（向左滑），后退返回 `+1`（向右滑）。新页面先停在对侧的屏幕外，然后两个页面沿同一个方向一起过渡它们的 `transform`：

```js
function getDir(prev,next){
  if(dirMode==='ltr')return 1; if(dirMode==='rtl')return -1; if(dirMode==='vertical')return 2;
  return next>prev ? -1 : 1;   // auto: forward = left(-1), back = right(+1)
}

function doTransition(prev,next,dur,stagger,ease){
  const oldEl=pages[prev], newEl=pages[next];
  const d=getDir(prev,next), dir=Math.abs(d)===2?'Y':'X', sign=d<0?-1:1;
  newEl.style.transition='none';
  newEl.style.transform=`translate${dir}(${sign*-100}%)`;   // park off-screen, opposite edge
  void newEl.offsetWidth;                                  // commit the parked position
  oldEl.style.transition=`transform ${dur}ms ${ease}`;
  newEl.style.transition=`transform ${dur}ms ${ease} ${stagger}ms`;
  oldEl.style.transform=`translate${dir}(${sign*100}%)`;   // both move the same way
  newEl.style.transform='translate(0,0)';
}
```

因为两个页面沿同一个方向、按同样的 100% 做 `transform` 动画，它们在移动时始终边对边锁在一起——合成器全权处理，这就是为什么每个页面都设置了 `will-change:transform`。「页面之间的间隔」设置给进入的页面加一个可选的延迟，让它跟在离开的页面后面。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 方向 | 前进和后退 | 前进和后退时，下一页向左滑、上一页向右滑；其他选择总是朝同一个方向滑 |
| 速度 | 普通 | 每次滑动需要多长时间：慢是 800ms，普通 500ms，快 300ms；300 到 500ms 符合手机应用，时间再长就拖沓 |
| 感觉 | 平滑 | 平滑减速到停；弹性会滑过头一点再回来；均匀保持一个稳定的速度 |
| 页面之间的间隔 | 无 | 新页面晚开始多久：无、小（80ms）或大（160ms）；延迟会在页面之间留出一段间隔 |

## 生产注意事项
- **用 `transform: translate` 做动画，永远不要用 `left`/`right`**——translate 在合成器上运行，能保持 60fps；位置属性每一帧都会触发布局。
- **确认停放位置。**把新页面停在屏幕外之后读一下 `offsetWidth`，会让浏览器在过渡开启前先应用那个位置；没有这一步，两次写入会合并，新页面会直接跳进来，没有滑动。
- **屏幕外的页面保持 `pointer-events` 禁用**，直到激活为止，这样停放的面板不会拦截点击。
- **方向要符合平台预期。**在移动端，前进向左、后退向右是肌肉记忆；反过来会让人迷失方向。「前进和后退」这个设置就是这么编码的。
- **减弱动态效果**会切换 `.active` 类并清除 transform，不做滑动。
- **等价的库**：View Transitions API 用 `::view-transition-old/new` 上的滑动关键帧表达这一点。Framer Motion 的 `AnimatePresence`（用 `x` 变体）、React Router 的过渡库和 GSAP 的 `xPercent` 补间都实现了同样的成对平移。

## 另见
- [View Transitions API](../view-transitions-api/) — 浏览器也能滑动页面
- [缩放过渡](../zoom-transition/) — 页面在纵深中移动而不是横向移动
- [交叉淡入淡出过渡](../crossfade/) — 页面淡入淡出而不是移动
- [弹性过渡](../elastic-transition/) — 带弹簧收尾的滑动
