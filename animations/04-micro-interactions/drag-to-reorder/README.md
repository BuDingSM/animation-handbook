# 拖动排序

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/drag-to-reorder/)

## 它是什么
拖动排序让人们拿起一个项目、移动它，来改变列表的顺序。项目抬到其他项目上方，跟随指针，经过的每个项目都会滑开，让出它要落下的空位；松开，它就落进那个空位。同样的操作可以用键盘完成：拿起项目，用方向键移动，再放下。

## 什么时候用
- 待办清单、播放列表和优先级排序，顺序本身就有意义
- 菜单、导航链接、表单字段或仪表盘小部件的编辑器
- 短列表，大约一屏以内，拖动比反复按“上移”更快
- 触屏优先的列表，只要用手柄或短按开始拖动，页面仍然能滚动

## 工作原理
每一行都放在列表顶部，用变换移到它的位置：`--i` 存着位置，样式表把它变成带过渡的 `translateY`，所以新顺序永远不会改变布局。拿着的行用内联变换跟随指针，没有过渡，经过的行得到新的 `--i`，于是滑开：

```js
// The rows between where the held one started and slot `to` slide one slot toward the gap it left.
function makeRoom(to){
  const {row,from}=drag;drag.to=to;
  order.forEach((r,j)=>{if(r!==row)place(r,from<to&&j>from&&j<=to?j-1:to<from&&j>=to&&j<from?j+1:j)});
}
// Pointer and Show me: the held row is drawn at y, exactly under the pointer, and the others make room around the slot it is over.
function follow(y){
  const s=slotPx(),n=order.length,r=drag.row;
  drag.y=Math.max(-.4*s,Math.min((n-.6)*s,y));
  r.style.transition='none';r.style.transform=`translateY(${drag.y}px)`;
  const to=Math.max(0,Math.min(n-1,Math.round(drag.y/s)));
  if(to!==drag.to)makeRoom(to);
}
```

松开时，`drop()` 清除内联变换，于是行从指针下方滑进它的位置，列表的真实顺序也跟上：经过的行用 `insertBefore` 移到它的另一边，这样拿着的行永远不离开文档，并保持键盘焦点。移动文档里的元素会结束它正在进行的过渡，所以还在滑开的行会跳到它的位置。`keepSlides()` 用 FLIP 的想法来对付这一点：它先读出每行画在哪里，移动后把那个位置作为静止变换还给它，然后放开，滑动就从那里继续。重置也是通过它把整个列表恢复顺序，于是行滑回家：

```js
function keepSlides(rows,move){
  const at=rows.map(drawnY);move();
  rows.forEach((r,i)=>{r.style.transition='none';r.style.transform=`translateY(${at[i]}px)`});
  void list.offsetWidth;rows.forEach(r=>{r.style.transition=r.style.transform=''});
}
```

抬起是行内卡片上的 `scale`，加伪元素上不透明度淡入的阴影，所以阴影永远不用做动画。

指针事件用一套代码路径覆盖鼠标、触控笔和触摸，`setPointerCapture` 让行即使指针离开它也能跟随。鼠标立刻抬起行。手指从手柄（`touch-action: none`）立刻抬起，从行的其他地方则要在按压 300ms 后才抬起，这样在列表上快速滑动仍然能滚动页面（行有 `touch-action: pan-y`）；一旦拿着一行，一个非被动的 `touchmove` 监听器会调用 `preventDefault`，浏览器就不会在它下面滚动。键盘上每个手柄都是一个按钮：空格键或 Enter 拿起任务、放下任务，方向键移动一个位置，Esc 放回去，礼貌的实时区域会大声读出每一步。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 滑动速度 | 普通 | 经过的项目滑开、放下的项目落定有多快：慢是 420ms，普通 220ms，快 120ms |
| 抬起多少 | 轻微 | 轻微让拿着的项目变大 2%，在下方 8px 处有一道柔和的阴影；强烈让它变大 6%、倾斜 1.5°，阴影加深到下方 18px 处 |
| 仅用手柄拖动 | 关 | 只有手柄能开始拖动；关时，整行都可以，触屏上短按之后开始 |

## 生产注意事项
- **只在放下时保存一次顺序。** 拖动期间用变换移动项目，只在项目放下时重写列表（并发送到服务器）；每次指针移动都重建列表会让它闪烁，还会丢失指针捕获。
- **不同高度的行**：演示的行高度相同，所以一个位置是固定距离。高度混合时，测量每一行，把目标位置之前各行的高度加起来，或者每次改变位置时在 DOM 里移动元素，用 FLIP 给跳跃做动画。
- **长列表**：拿着的项目靠近容器顶部或底部边缘时滚动容器；列表超过一屏时，提供“移到顶部”或位置输入框，因为长拖动很累、容易提前掉落。
- **无障碍**：给每个项目一个用键盘移动的办法，在实时区域里播报拿起、每次移动、放下和取消，并让焦点留在被移动的项目上。绝不要只靠拖动。
- **减弱动态效果**：保留重排，去掉滑动；`prefers-reduced-motion: reduce` 下项目直接跳到它们的位置上，键盘方式本来就是每次按键移动一个位置。
- **库的等价物**：纯 JavaScript 用 SortableJS（它的 `handle` 和 `delay` 选项对应这里的开关和触屏按压），React 用 dnd-kit 的 sortable 预设和 @hello-pangea/dnd（两者都会向读屏软件播报移动），Framer Motion 的 `Reorder.Group` 和 `Reorder.Item`，想要自己的物理效果时用 GSAP 的 Draggable。

## 另见
- [滑动删除](../swipe-to-dismiss/) — 移除项目的横向拖动
- [下拉刷新](../pull-to-refresh/) — 向下拉动、重新加载列表
- [FLIP 技巧](../../03-page-transitions/flip-technique/) — 顺序变化时滑到新位置的卡片
