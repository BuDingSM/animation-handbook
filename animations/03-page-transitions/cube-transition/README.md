# 立方体过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/cube-transition/)

## 它是什么
立方体过渡把两个页面当作立方体相邻的两个面。页面切换时，立方体转四分之一圈：旧页面绕立方体的中线转走，新页面从侧面转进来，取代它的位置。返回时立方体朝反方向转，所以访客总能知道自己往哪个方向走了。

## 什么时候用
- 照片故事和产品导览，每页是一张大图，旋转增加穿过空间的感觉
- 有明确顺序的幻灯片和新手引导步骤
- 手机上的故事格式，故事之间横向的立方体旋转已经很熟悉
- 短序列的页面；效果很强，所以适合少数几次切换，而不是应用里的每次点击

## 工作原理
没有真正建一个立方体。旋转期间，两个页面得到同样的 3D 变换：后移到立方体的中线（屏幕后半个边长），旋转，再前移回来。旧页面从 0 转到 90 度，新页面从 90 度转到 0，所以两者总是在立方体的边上相接。页面区域给场景提供透视，每个页面通过一个黑色图层变暗，图层的不透明度跟随旋转：

```js
const side=axis==='Y',size=side?area.clientWidth:area.clientHeight,half=(size/2).toFixed(1),d=dur*(slowTog.checked?3:1);
const end=(next>prev?1:-1)*(side?-90:90);
const face=a=>`translateZ(-${half}px) rotate${axis}(${a}deg) translateZ(${half}px)`;
const how={duration:d,easing:'cubic-bezier(.65,0,.35,1)',fill:'forwards'};
area.style.perspective=Math.round(size*depth)+'px';
oldEl.animate([{transform:face(0)},{transform:face(end)}],how);
newEl.animate([{transform:face(-end)},{transform:face(0)}],how);
dims[prev].animate([{opacity:0},{opacity:.65}],how);
dims[next].animate([{opacity:.65},{opacity:0}],how);
```

两个页面都有 `backface-visibility: hidden`。旋转开始和结束附近，一个面背对观众，真实的立方体不会显示它；没有这条规则，它的镜像背面可能被画在另一个页面上面。页面区域裁剪内容（`overflow: hidden`），让它保持平坦：两个页面按页面顺序绘制，不按深度排序。只有 `transform` 和 `opacity` 在变，所以浏览器可以在图形芯片上跑完整次旋转。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 旋转方向 | 横向 | 横向绕一条竖线旋转，更晚的页面从右侧进来；上下绕一条横线旋转，更晚的页面从下面进来 |
| 速度 | 普通 | 立方体旋转需要多长时间：慢是 1100ms，普通 750ms，快 500ms |
| 深度 | 普通 | 观众离立方体有多近：浅让他们离 3.2 个边长，普通 1.6，深 0.9，所以近边看起来更大 |

## 生产注意事项
- **按页面尺寸定立方体。**立方体的深度与它转过的边一样，所以在旋转开始时读页面的宽（或高）；固定深度会让两个页面在其他屏幕尺寸上在边缘处分开或重叠。
- **藏起背面。**没有 `backface-visibility: hidden`，一个转过观众的面会显示它的镜像背面，没有深度排序时它可能被画在前面的页面上面。
- **用阴影，不要用模糊。**一个不透明度跟随旋转的黑色图层能卖出光线衰减的感觉；模糊或盒阴影会每一帧都重绘。
- **注意晕动。**一次大的 3D 旋转是很多运动。减弱动态效果改为在 300ms 内淡入新页面，旋转应该保持短暂。
- **等价的库。**Swiper 的立方体效果、View Transitions API（在带透视的组里给 `::view-transition-old` 和 `::view-transition-new` 用 3D 关键帧）以及 GSAP 带 `transformPerspective` 的 `rotationY` 补间，都构建了同样的四分之一圈旋转。

## 另见
- [滚动驱动 3D 旋转](../../06-3d-advanced/scroll-driven-3d-rotation/) — 一个由滚动旋转的 3D 立方体
- [3D 翻转卡片](../../06-3d-advanced/flip-card-3d/) — 一张卡片翻过来显示背面
- [滑动过渡](../slide-transition/) — 扁平版本：页面横向滑动
- [翻页卷角](../page-curl/) — 页面像纸一样翻过去
