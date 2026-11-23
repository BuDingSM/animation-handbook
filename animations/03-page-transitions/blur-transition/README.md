# 模糊过渡

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/blur-transition/)

## 它是什么
模糊过渡让旧页面失焦直到消失，再把新页面从模糊带到清晰。看起来像相机从一个主体拉焦到另一个主体，所以切换读起来像是注意力的转移，而不是一次硬切。

## 什么时候用
- 视觉丰富的页面之间的过渡，硬切显得突兀
- 摄影、电影和编辑类网站，拉焦的比喻与内容相符
- 弹窗或详情视图，应该感觉像背景退到焦外
- 只用于用户主动发起的导航——绝不用在自动播放的轮播上，那里的成本会累积

## 工作原理
离开的页面把 `filter`（以及可选的 `opacity`）过渡到最大模糊，然后进入的页面预先模糊、设为激活、再过渡回 `blur(0px)`。因为滤镜过渡无法跨元素交换合成，进入页面的模糊先以 `transition: none` 设置，强制回流确认，再做动画变清晰：

```js
function doTransition(prev, next, dur, ease, maxBlur) {
  const o = pages[prev], n = pages[next];
  const withFade = fadeTog.checked, overlap = overlapTog.checked;
  const t = `filter ${dur}ms ${ease}` + (withFade ? `, opacity ${dur}ms ${ease}` : '');
  o.style.transition = t;
  o.style.filter = `blur(${maxBlur}px)`;
  if (withFade) o.style.opacity = '0';

  const swapAfter = overlap ? Math.round(dur * 0.5) : dur;
  setTimeout(() => {
    o.classList.remove('active'); o.style.opacity = '0';
    n.style.transition = 'none';
    n.style.filter = `blur(${maxBlur}px)`; n.style.opacity = '0'; n.classList.add('active');
    void n.offsetWidth;                                        // commit the blurred start
    n.style.transition = t;
    n.style.filter = 'blur(0px)'; n.style.opacity = '1';   // sharpen in
  }, swapAfter);
}
```

「两半重叠」设置让进入的变清晰在离开的模糊进行到 50% 时就开始，而不是等它结束，在两半交叉的同时把总时间缩短约四分之一。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 模糊程度 | 中等 | 页面在中途模糊到什么程度：轻微是 8px，中等 20px，强烈 35px；超过约 30px 后对浏览器来说很重，还可能看起来浑浊 |
| 速度 | 普通 | 每一半需要多长时间，先模糊出去再变清晰进来：慢是 650ms，普通 400ms，快 250ms |
| 边模糊边淡入 | 开 | 每页在模糊时也淡入淡出，这样能遮住换页；关时页面保持实色、只模糊 |
| 两半重叠 | 关 | 在模糊进行到一半时就开始让新页面变清晰，这样切换时间缩短四分之一 |
| 感觉 | 轻柔 | 轻柔感觉像受控的拉焦；平滑更快地进入焦点；均匀保持一个稳定的速度 |

## 生产注意事项
- **GPU 成本。**在全视口元素上做大半径 `filter: blur()` 动画是成本最高的操作之一。让模糊区域尽量小，在移动端限制半径，滚动时不要运行它。
- **确认起始状态。**用 `transition: none` 设置预模糊、再打开过渡，中间需要一次强制回流（读 `offsetWidth`）；没有它，浏览器会合并两次写入，页面直接以清晰状态弹出。
- **边缘泛光。**模糊一个有硬直角边缘的元素时，模糊在边界外采样，可能会露出一圈淡淡的光晕。一个微妙的 `overflow: hidden` 容器或轻微的缩放能让边缘干净。
- **等价的库。**View Transitions API 可以在 `::view-transition-old`/`-new` 上做 `filter` 动画，达到同样的效果，记账工作少得多。Framer Motion 通过 `animate` 把 `filter: 'blur(20px)'` 做到 `'blur(0px)'`；GSAP 通过它的 CSS 插件做同样的事。Barba.js 的 `leave`/`enter` 钩子承载模糊出去和变清晰进来的两半。

## 另见
- [交叉淡入淡出过渡](../crossfade/) — 模糊之下的普通淡入淡出
- [闪光 / 漏光过渡](../flash-transition/) — 一道闪光藏住切换
- [溶解过渡](../dissolve/) — 瓷砖藏住切换
- [缩放过渡](../zoom-transition/) — 缩放转移注意力而不是焦点
