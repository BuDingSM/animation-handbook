# 滚动吸附

[在线演示](https://animation.xizanghuapi.cn/animations/01-scroll-based/snap-scrolling/)

## 它是什么

滚动吸附让滚动区域停在设定的点上，而不是停在任何地方。停止滚动时，浏览器把视图拉到最近的区域，于是每个区域都完整显示，像翻页一样。拉动由浏览器自己完成，不需要脚本。

## 什么时候用

- 落地页，其中每个区域都是一块完整的信息单元，局部视图会显得别扭
- 移动优先的演示，滑动分页符合原生应用的惯例
- 幻灯片、产品画廊和功能导览，步骤数量明确
- 任何中途停住的滚动位置感觉像是意外而不是有意的布局

## 工作原理

三个 CSS 属性实现了整个机制：

```css
/* On the scroll container */
.scroller {
  overflow-y: auto;
  container-type: size;          /* 100cqh is the box's height */
  scroll-snap-type: y mandatory; /* or 'y proximity' */
}

/* On each section */
.section {
  height: 100cqh;                /* one box tall */
  scroll-snap-align: start;
}
```

`mandatory` 在每次滚动结束时强制吸附。`proximity` 只在用户靠近吸附点时吸附。`none` 禁用吸附。“吸附”设置通过三个 CSS 规则读取的 `data-snap` 属性在三者之间切换。“播放”按钮滚动盒子时，一条共享规则会关掉它的吸附，停下后页面的吸附重新生效。

圆点滚动到它对应的区域；不写 `behavior` 就让盒子自己的 `scroll-behavior` 决定，于是它滑动，或在减弱动态效果下跳过去：

```js
scroller.scrollTo({ top: sectionIndex * scroller.clientHeight });
```

当前区域由滚动位置推导：

```js
const sectionIndex = Math.round(scroller.scrollTop / scroller.clientHeight);
```

## 关键参数

| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 吸附 | 总是 | 每次滚动后都停在一个区域上；靠近时只在盒子停在某个区域附近才停下；关从不吸附 |

## 生产注意事项

- **CSS 就是整个机制。**滚动吸附本身不需要 JavaScript。JS 只用于辅助界面——分页指示器、跳转按钮，或把外部状态同步到当前区域。
- **`mandatory` 与 `proximity`。**每个区域都是视口高、从不希望出现局部视图时用 `mandatory`。区域高度不一，或只希望在有意的手势下才吸附时用 `proximity`。`mandatory` 配小步滚动（键盘方向键）会感觉卡住、很生硬。
- **Mobile Safari。**iOS Safari 历史上在溢出容器内使用 `scroll-snap-type` 时有问题。在真机上测试。变通办法包括用 `overflow: auto` 代替 `scroll`。
- **不需要库。**与本手册大多数其他条目不同，滚动吸附除了浏览器什么都不需要。这个功能是原生的，在所有现代浏览器中都得到良好支持。
- **无障碍。**滚动吸附可能让期待连续滚动的键盘用户感到困扰。确保 `Tab` 键能在区域之间导航，区域标题可以获得焦点，并且有跳过导航的链接。

## 另见

- [层叠卡片](../stacking-cards/) — 卡片滚动时叠成一摞
- [区域擦除切换](../section-wipe/) — 每个区域从上一个上面滑过
- [联系人列表飞入飞出](../fly-in-fly-out-contact-list/) — 行在靠近边缘时淡入淡出、滑入滑出
- [3D 轮播](../../06-3d-advanced/carousel-3d/) — 卡片排成 3D 圆环，一次停在一张卡片上
