# 变形过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/morph-transition/)

## 它是什么
变形过渡通过把轮廓上的每个点移向新形状里对应的点，把一个形状变成另一个形状。不是换图片，而是轮廓在流动：圆形展开成六边形，六边形收尖成星形。在演示里，一个小 Logo 会随着你在首页、画廊和关于之间移动而改变形状。

## 什么时候用
- Logo 或品牌标志，靠变形来提示分区变化
- 图标状态变化（播放↔暂停、菜单↔关闭、勾选出现）
- 几何形状本身带有含义的数据形状过渡
- 共享一个持久元素的页面之间的装饰性连续感

## 工作原理
变形要求源路径和目标路径的**点数相同、顺序相同**。演示把每个形状归一化为 12 个顶点，然后在每一帧动画里，按缓动后的进度对每个 `[x,y]` 点做线性插值，再重建路径字符串：

```js
function lerp(a,b,t){ return a+(b-a)*t }

function startMorph(from,to,done){
  const start=performance.now();
  function tick(now){
    const t=Math.min((now-start)/dur,1), et=easeFn(t);
    const pts=from.map(([ax,ay],i)=>[ lerp(ax,to[i][0],et), lerp(ay,to[i][1],et) ]);
    morphPath.setAttribute('d', pts2path(pts));
    if(t<1) morphRaf=requestAnimationFrame(tick); else done?.();
  }
  morphRaf=requestAnimationFrame(tick);
}
```

这些形状是按保证顶点数匹配的方式生成的：圆形在半径周围取 12 个点，六边形把 6 个角和 6 个边的中点交错排列，星形把 6 个外点和 6 个内点交替排列。因为源形状的第 `i` 个点对应目标形状的第 `i` 个点，每个点都有明确的目的地，轮廓永远不会撕裂。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 速度 | 普通 | 形状变化需要多长时间：慢是 1100ms，普通 700ms，快 400ms；越慢读起来越郑重 |
| 感觉 | 轻柔 | 轻柔先慢后快再慢；平滑减速到停；弹性会拉伸得比新形状稍大一点再回来；均匀保持一个稳定的速度 |

## 生产注意事项
- **点数和顺序就是一切。**如果两条路径的顶点数不同，插值前必须把其中一条重采样到匹配——不匹配会产生垃圾结果。flubber 这类工具能自动插入和配对点来解决这个问题。
- **朴素的 `lerp` 会切掉拐角**，对于带曲线的路径尤其如此。这个演示在点之间用直的 `L` 线段；对贝塞尔曲线做变形时，还需要同时插值控制手柄。
- **`requestAnimationFrame` + `performance.now()`** 在这里手动驱动计时，而不是用 CSS，因为 CSS 无法对 `d` 属性的点列表在任意形状之间做插值。
- **减弱动态效果**会跳过变形，直接换页，标志留在目标形状。
- **等价的库**：GSAP 的 MorphSVG 插件和独立的 flubber 库能处理任意路径的点重采样和配对。Framer Motion 可以给简单的 SVG `path` 值做动画；Lottie 则烘焙从 After Effects 导出的形状变形。

## 另见
- [共享元素过渡](../shared-element-transition/) — 一个元素移动而不是变形
- [FLIP 技巧](../flip-technique/) — 元素滑到新位置
- [交叉淡入淡出过渡](../crossfade/) — 承载页面内容的淡入淡出
- [弹性过渡](../elastic-transition/) — 冲过头再落定的弹簧
