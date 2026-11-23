# 3D 轮播

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/carousel-3d/)

## 它是什么
3D 轮播把卡片围成一圈，整圈转动，把一张卡片带到前面。每张卡片按圆周的等份转动，再从中间向外推出同样的距离，所以前面卡片旁边的会向外倾斜，后面的会露出背面。拖动能转动圆环，松手后它会滑行一小段，再停在最近的卡片上。

## 什么时候用
- 图片画廊和作品集，浏览起来应该有趣
- 只有少量选项的产品选择器，比如颜色、型号或套餐
- 一次展示一张的专辑封面、书封或活动海报
- 卡片数量不多（约 5 到 12 张）的落地页展示

## 工作原理
四层嵌套元素承载 3D。盒子有 `perspective`（镜头）；里面一个包装层把圆环按半径向后推，另一个为从上方看而倾斜它；圆环本身负责转动。每一层都用 `preserve-3d` 让子元素保持在 3D 里，每张卡片的正反两面在背对时隐藏：

```css
.view { perspective: 900px; }
.push { transform-style: preserve-3d; transform: translateZ(calc(var(--r) * -1)); }
.tilt { transform-style: preserve-3d; transform: translateY(calc(var(--r) * -0.11)) rotateX(-14deg); }
.ring { transform-style: preserve-3d; }   /* the script sets rotateY() */
.face { backface-visibility: hidden; }    /* .back is turned 180° */
```

卡片 `i` 转动 `i × 360° / N`，再按半径向外推出。N 张带小间隙的卡片组成一个正多边形，从中心到一边中点的距离是半边长除以 tan(180° / N)。把圆环按同样的半径向后推，前面的卡片就保持正常大小：

```js
const step = 360 / N;
const R = cardWidth * 1.12 / (2 * Math.tan(Math.PI / N));   // a gap of 12% of a card
card.style.transform = `rotateY(${i * step}deg) translateZ(var(--r))`;
ring.style.transform = `rotateY(${angle}deg)`;               // angle -i * step brings card i to the front
```

拖动按圆环的半径把像素换成角度，所以前面的卡片会跟着指针走：`angle = startAngle + dx × 180 / (π × R)`。Pointer Events 同时服务鼠标、触摸和手写笔，`touch-action: pan-y` 让竖向滑动仍然能滚动页面。松手时，最后一小段时间的速度决定圆环会滑行多远；那个位置四舍五入到最近的卡片，圆环按流逝的时间缓动过去，所以在任何帧率下速度都一样：

```js
const GLIDE = FRAME / -Math.log(1 - 0.09);            // the easing's time constant, about 177 ms
const throwDeg = speedPxPerMs * GLIDE * 180 / (Math.PI * R);
target = Math.round((angle + throwDeg) / step) * step;
// every frame
angle += (target - angle) * (1 - Math.pow(1 - 0.09, dt / FRAME));
```

因为抛出的距离是指针速度乘以缓动的时间常数，缓动以指针的速度开始，滑行从拖动处无缝接续。几乎没动的一按会把按中的卡片带到前面。箭头和方向键一次移动一张卡片，快速连按会累加。「演示一下」让圆环在三秒里整整转一圈，先快后慢，滑行停在出发的那张卡片上。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 卡片数量 | 8 | 圆环里站着几张卡片：6、8 或 12 张，相邻卡片之间留一张卡片 12% 的间隙。卡片越多圆环越宽；装不下时，比如手机上的 12 张卡片，卡片会画得小一些，保持在盒子宽度的 94% 以内 |
| 从上方看 | 开 | 把圆环向后倾斜 14 度，后面的卡片会从前面卡片的上方露出来；关掉就正面直视圆环 |
| 自己旋转 | 关 | 圆环每秒转 12 度，拖动时等你，停稳后过一会儿再开始；减弱动态下改为每三秒走一张卡片 |

## 生产注意事项
- **无障碍**：给圆环一个名字和角色（`role="group"` 加 `aria-roledescription="carousel"`）、真正的上一个/下一个按钮，以及获得焦点时圆环上的方向键。通过礼貌的 live 区域播报停在前面的卡片，但圆环自己转动时不要播报。
- **自己动**：任何自己动超过五秒的东西都需要停下来的办法（WCAG 2.2.2）。指针悬停在圆环上或它获得键盘焦点时暂停旋转，并提供一个暂停按钮。
- **减弱动态**：在 `prefers-reduced-motion: reduce` 下，从一张卡片跳到下一张，而不是转动。演示每次拖动、按箭头或按键走一张卡片，「演示一下」走到下一张再走回来。
- **性能**：移动时只有圆环的 `transform` 在变，所以浏览器合成卡片而不用重新绘制。带 `preserve-3d` 的元素上有 `filter`、`overflow: hidden` 或小于 1 的 `opacity` 会把它压平，圆环会塌成一张平面图。
- **库**：Swiper 的 coverflow 和创意效果、GSAP Draggable 加 InertiaPlugin 的抛掷，以及 Framer Motion 带惯性的 `drag`，都能做出同样的东西。CSS 3D 不够时，Three.js 能给出带光照和反射的真正 3D 场景。

## 另见
- [3D 翻转卡片](../flip-card-3d/) — 一张卡片在 3D 中翻过来露出背面
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜
- [滚动吸附](../../01-scroll-based/snap-scrolling/) — 一次停在一张卡片上的平面画廊
- [滑动删除](../../04-micro-interactions/swipe-to-dismiss/) — 一张卡片跟着拖动走，甩动后飞走
