# 覆盖擦除

[在线演示](https://animation.xizanghuapi.cn/animations/03-page-transitions/overlay-wipe/)

## 它是什么
覆盖擦除把页面切换藏在一块纯色面板后面。面板从一侧扫进来，直到盖住整个页面，趁没人能看见时换页，再从对侧扫出去，露出新页面。第二块浅色面板可以在进入时跑在它前面一点、离开时跟在它后面一点，这样边缘会显示出两条色带。

## 什么时候用
- 作品集和代理公司网站，希望每次页面切换都感觉经过设计、符合品牌
- 页面加载慢一拍的网站：盖住的瞬间藏住了等待
- 故事或产品导览里的章节分隔，下一部分的名字可以放在面板上
- 任何旧页面和新页面之间没有共同点可供动画的切换

## 工作原理
每块面板都是页面区域上方一个绝对定位的盒子，每次滑动是一次 Web Animations 对 `transform` 的调用，结束时保持在结束位置。切换分三步，每一步在前一步完成后开始：面板滑入；当前面板滑入完成后，换掉它后面的页面；只有那时面板才滑出，在一段短暂的盖住之后，当最后一块离开视野时它们被拿走：

```js
function slide(el,from,to,ms,wait){
  return el.animate([{transform:from},{transform:to}],{duration:ms,delay:wait,easing:EASE,fill:'both'});
}
// in navigate():
const ms=sweepMs*(slowTog.checked?3:1),stay=ms/4,two=twoTog.checked,lead=two?ms/4:0,from=FROM[dir],to=TO[dir];
if(two)slide(back,from,'none',ms,0);              // the lighter panel goes in first
slide(front,from,'none',ms,lead).onfinish=()=>{    // 1. in: the page is now covered
  rest(oldEl,false);rest(newEl,true);              // 2. swap behind the panel
  const out=slide(front,'none',to,ms,stay),last=two?slide(back,'none',to,ms,stay+lead):out;
  last.onfinish=()=>{clearWipe();finish()};        // 3. out: the lighter panel leaves last
};
```

面板在合成器上移动，换页在主线程上运行。由计时器启动的换页在主线程繁忙时会迟到，面板可能已经离开；像这样串起来，繁忙的时刻只会让盖住的瞬间变长。`FROM` 和 `TO` 保存每个方向的两个屏幕外位置（对于从左到右，`translateX(-100%)` 和 `translateX(100%)`），所以面板总是从与进来那侧相反的一侧离开。缓动 `cubic-bezier(.7,0,.3,1)` 让每次扫动起步和停下都柔和、中间快速通过。只有 `transform` 在变，所以浏览器移动面板时不用布局或重绘页面。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 方向 | 从左到右 | 面板从哪一侧进来；它总是从对侧离开 |
| 面板颜色 | 紫色 | 面板的颜色；第二块面板是它的浅色 |
| 两层叠加面板 | 关 | 一块浅色面板在进入时跑在主面板前面一点，离开时跟在它后面一点 |
| 速度 | 普通 | 每次扫过需要多长时间：慢是 550ms，普通 400ms，快 270ms；页面被盖住的时间是它的四分之一 |

## 生产注意事项
- **滑动，永远不要调整尺寸。**增大面板的宽或高会让浏览器每一帧都布局页面；`transform` 移动的是一个已经画好的图层。
- **只在盖住时换页，并把步骤串起来。**在面板完全盖住之前换页，会透过缝隙露出一道切口；由计时器定时的换页如果主线程卡住，可能在面板已经离开之后才发生。在滑入完成时换页，再从那里开始滑出。
- **真实的页面加载。**在多页面网站上，离开页面前播放前一半（进入），在新页面上播放后一半（离开）；View Transitions API 可以用 `::view-transition-old` 和 `::view-transition-new` 以及一个作为面板的伪元素同时做这两半。Barba.js 的 `leave` 和 `enter` 钩子对应同样的两半。
- **保持短暂。**在整天使用的应用里每次点击都擦除一次会让人疲倦；整个切换在一秒以内是个好的上限。
- **减弱动态效果**跳过面板，在 250ms 内淡入新页面。
- **等价的库。**GSAP 时间线（用 `xPercent` 补间加一个换页回调）、Framer Motion 的 `AnimatePresence`（带一个覆盖层 `motion.div`）以及 Swup 或 Barba.js 的页面过渡钩子都能构建这种模式。

## 另见
- [幕布揭示](../../02-entrance-and-exit/curtain-reveal/) — 揭示单个元素的面板
- [区域擦除切换](../../01-scroll-based/section-wipe/) — 滚动时各部分互相滑过
- [闪光 / 漏光过渡](../flash-transition/) — 一道闪光藏住切换
- [滑动过渡](../slide-transition/) — 页面自己滑动而不是面板
