# View Transitions API

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/view-transitions-api/)

## 它是什么
View Transitions API 是浏览器自带的功能，帮你给页面切换做动画。你告诉浏览器页面要变了；它在变化前后各给页面拍一张照，然后在两张照片之间做动画。照片怎么动由 CSS 决定，所以同一次点击可以淡入淡出、滑动、缩放或倾斜。

## 什么时候用
- SPA 里的同文档导航，在原地替换页面内容
- 希望在完整文档之间有原生感过渡的多页面导航（用跨文档变体）
- 前后 DOM 很复杂、手动给每个元素做动画容易出错的情况
- 希望过渡样式是 CSS 的事、与导航逻辑解耦的情况

## 工作原理
把 DOM 改动包进 `startViewTransition()`。浏览器截取当前帧，执行回调，截取新帧，再在两者之间做动画。一个命名区域——用 `view-transition-name` 设置——会被当作自己的一对快照来跟踪，CSS 伪元素分别给旧快照和新快照做动画：

```js
async function navigate(next){
  if(next===current||animating)return;
  animating=true;current=next;
  document.documentElement.dataset.vt = style==='crossfade' ? '' : style;
  document.documentElement.style.setProperty('--vt-dur', dur+'ms');
  if(supportsVT){
    const t = document.startViewTransition(()=>showPage(current));
    await t.finished;
  }else{
    /* manual opacity fallback */
  }
  animating=false;
}
```

```css
.page-area{ view-transition-name: page-content; }
/* Suppress the default root cross-fade; only page-content animates */
::view-transition-old(root),::view-transition-new(root){ animation:none }
::view-transition-old(page-content){ animation:vt-out var(--vt-dur) var(--vt-ease) both }
::view-transition-new(page-content){ animation:vt-in  var(--vt-dur) var(--vt-ease) both }
@keyframes vt-out{ from{opacity:1} to{opacity:0} }
@keyframes vt-in { from{opacity:0} to{opacity:1} }
```

切换 `data-vt` 属性会换入另一组关键帧（滑动、缩放或轻微倾斜），所以一条代码路径能做出四种不同的过渡；速度和感觉设置的是关键帧读取的 `--vt-dur` 和 `--vt-ease` 自定义属性。`t.finished` 在动画完成时兑现，演示会等它完成才允许再次进入。更新回调在运行时展示当前页面的静止状态，所以一次被跳过的过渡（按重置，或再次按「演示一下」）不会带回旧页面。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 切换样式 | 淡入淡出 | 淡入淡出混合两张照片；滑动把旧照片向左推出去，同时新照片从右侧进来，各移动一个完整宽度；缩放把旧照片缩小到 80% 并淡出，同时新照片淡入并从稍大的 115% 落定；倾斜让两者在淡入淡出时都微微转动：旧照片缩小到 90% 并逆时针转 2°，新照片从 90%、顺时针转 2° 开始 |
| 速度 | 普通 | 切换需要多长时间：慢是 800ms，普通 500ms，快 300ms；低于约 150ms 时看起来像瞬间切换 |
| 感觉 | 轻柔 | 轻柔先慢后快再慢；平滑减速到停；均匀保持一个稳定的速度 |

## 生产注意事项
- **做特性检测**：用 `'startViewTransition' in document` 检测，没有 API 时回退到手动的不透明度交叉淡入淡出——演示正是这么做的，所以不支持 API 的浏览器仍然能得到一个简单的 300ms 淡入淡出，无论切换样式和速度是什么。
- **浏览器支持。**同文档视图过渡——像这个演示里的 `document.startViewTransition()`——支持 Chrome 和 Edge 111 及更高版本、Safari 18 和 Firefox 144。跨文档过渡，用 `@view-transition { navigation: auto; }` 为完整页面加载开启，支持 Chrome 和 Edge 126 以及 Safari 18.2；Firefox 的当前支持情况请自行确认。旧浏览器要保留回退。
- **关掉根动画**（`::view-transition-old(root)`），当你只想让一个子区域动的时候；否则整个页面会在你的命名区域下面交叉淡入淡出。
- **每个 `view-transition-name` 在截取时必须唯一**。同一张快照里两个元素共用一个名字会让浏览器跳过过渡：页面照样换，只是没有动画。
- **尊重减弱动态效果**——当设置了 `prefers-reduced-motion: reduce` 时，演示直接用 `showPage()`，不做动画。
- **按压要等过渡结束。**视图过渡运行时，Chrome 会把页面上的每一次按压都发给根元素，所以在过渡结束前按钮不会响应（过渡伪元素上的 `pointer-events: none` 在根元素被截取时——默认就是这样——不起作用）；让页面过渡保持短暂。演示会把访客在方框上的按压当作接管，所以点那里仍然能停下「演示一下」。
- **等价的库**：Astro 的 `<ClientRouter />`（Astro 5 之前叫 `<ViewTransitions />`）和 React 的 `<ViewTransition>` 组件（Next.js 的 App Router 支持它）为客户端页面切换封装了这个 API；完整页面加载之间，CSS 规则 `@view-transition { navigation: auto; }` 不用 JavaScript 就能做到。Barba.js 和 Swup 比这个 API 更早，自己给旧页面和新页面内容做动画；在支持的浏览器上，你常常不再需要它们。

## 另见
- [交叉淡入淡出过渡](../crossfade/) — 同样的淡入淡出，手工实现
- [滑动过渡](../slide-transition/) — 页面滑动，带有方向感
- [缩放过渡](../zoom-transition/) — 三种页面间缩放方式
- [共享元素过渡](../shared-element-transition/) — 一张图片长进下一页
