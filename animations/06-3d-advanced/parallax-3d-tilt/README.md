# 视差 3D 倾斜

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/parallax-3d-tilt/)

## 它是什么
3D 倾斜让一张平面卡片朝指针倾斜，好像你从不同角度看一张真实的卡片。卡片的容器给场景带来景深，一道柔光跟着指针滑过卡面，补全了这个错觉。浏览器自己的 3D 变换就能做到，不需要 3D 库。

## 什么时候用
- 功能卡片、价格卡片和主视觉卡片，景深传递“高端”的感觉
- 悬停表示可选的作品集缩略图
- 任何标准平面悬停显得太被动的互动卡片
- NFT 和游戏界面，触感隐喻与产品相符

## 工作原理
`perspective` 必须设在**父元素**上，而不是卡片上。卡片的 `rotateX`/`rotateY` 由光标相对卡片中心的位置算出，归一化到 ±0.5，再乘以 MAX，所以卡片在边缘处倾斜 MAX/2。

绑定 **pointer** 事件，而不是 mouse 事件——同一个处理函数就能覆盖鼠标、手写笔和触摸：

```js
card.addEventListener('pointermove', e => {
  const r = card.getBoundingClientRect();
  const x = (e.clientX - r.left) / r.width  - 0.5;  // -0.5 to 0.5
  const y = (e.clientY - r.top)  / r.height - 0.5;

  const MAX = 15; // degrees
  const rx = (-y * MAX).toFixed(2);  // mouse up = card tilts toward viewer
  const ry = ( x * MAX).toFixed(2);  // mouse right = card tilts right

  card.style.transform = `rotateX(${rx}deg) rotateY(${ry}deg) scale(1.04)`;
});

card.addEventListener('pointerleave', () => {
  card.style.transform = 'rotateX(0deg) rotateY(0deg) scale(1)';
});
```

**光泽高光**——一个独立图层上的固定径向渐变，用变换移到光标处。每次移动都重写渐变会重绘整张卡片；移动已经画好的图层不会，卡片的 `overflow: hidden` 会裁剪它：

```css
.shine {                              /* --shine-d: twice the card's diagonal, set once by the script */
  position: absolute; left: 0; top: 0;
  width: var(--shine-d); height: var(--shine-d);
  margin: calc(var(--shine-d) / -2) 0 0 calc(var(--shine-d) / -2);   /* its middle starts on the card's top left corner */
  background: radial-gradient(circle closest-side, rgba(255,255,255,0.18) 0%, transparent 65%);
}
```

```js
// W and H: the card's width and height. D: --shine-d. R: how far the light reaches, out to the farthest corner
const R = Math.hypot((0.5 + Math.abs(x)) * W, (0.5 + Math.abs(y)) * H);
shine.style.transform = `translate(${(x + 0.5) * W}px, ${(y + 0.5) * H}px) scale(${2 * R / D})`;
```

**回正过渡**——只在离开时应用 `transition`，跟踪时不应用：

```js
card.addEventListener('pointermove',  () => card.classList.add('active'));
card.addEventListener('pointerleave', () => card.classList.remove('active'));
```

```css
.card { transition: transform 400ms ease; }
.card.active { transition: none; }  /* instant tracking while the pointer moves */
```

**「演示一下」**——演示的「演示一下」按钮用一个假指针而不是真指针移动同一张卡片。指针处理函数的工作是一个 `tilt(x, y)` 函数，接受 -0.5 到 0.5 的两个数字，「演示一下」每帧调用它一次，持续三秒，绕卡片中间转一圈半，最后停在那里：

```js
function step(now) {                           // t0 = the time of the first frame after Show me is pressed
  const p = Math.min(1, (now - t0) / 3000);    // progress, 0 to 1
  const r = 0.4 * Math.sin(Math.PI * p);       // distance from the middle: out to 0.4 and back
  const a = 3 * Math.PI * p;                   // angle: one and a half turns
  tilt(x0 * (1 - p) + r * Math.cos(a), y0 * (1 - p) + r * Math.sin(a));
  if (p < 1) requestAnimationFrame(step);
  else onLeave();                              // the card settles flat
}
```

`x0` 和 `y0` 是按下「演示一下」时卡片已经倾斜的位置，所以第二次按下会从那里继续。舞台上的一按、一根手指或一次滚轮转动，或指针移过卡片，都会取消那一帧并让卡片放平，所以控制权一直在用户手里。

卡片也可以获得键盘焦点，方向键通过同一个倾斜函数代替指针。键盘焦点让卡片朝它倾斜的方向倾斜（静止时只抬起，光在中间），每按一次方向键，假指针就向该方向移动卡片宽度的 6%（按住会重复移动），按 Escape 或把焦点移走会让它放平，和指针离开时一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 倾斜幅度 | 中等 | 卡片在边缘处倾斜多远：小是 4°，中等 7.5°，大是 12.5°；再大就开始感觉不稳了 |
| 3D 景深 | 中等 | 观看者看起来有多近：轻微是 1500px，中等 1000px，强烈 500px；越近倾斜越夸张 |
| 光泽 | 开 | 一道柔光跟着指针滑过卡片，像光落在真实卡片上 |
| 回正速度 | 普通 | 指针离开后卡片回到平放要多长时间：慢是 650ms，普通 400ms，快是 250ms |
| 朝指针反方向倾斜 | 关 | 改为让卡片朝指针的反方向倾斜，像被按下去一样 |
| 微微变大 | 开 | 倾斜时把卡片放大 4%，读起来像把它朝你抬起来 |

## 生产注意事项
- **触摸设备**：用 Pointer Events（`pointerdown`/`pointermove`/`pointerleave`）加 `touch-action: none`，触摸上免费获得效果——拖动倾斜卡片，轻触设置一次。不要用 `@media (hover: hover)` 来限制效果：混合触屏笔记本匹配它，配了鼠标的手机也可能匹配它，所以按媒体查询分支会在真实设备上杀死交互。如果必须分支，按实时的 `event.pointerType` 分支。
- **`will-change: transform`**：有很多倾斜卡片的页面上，只在指针悬停在卡片上时才加上它（`pointerenter`/`pointerleave`），避免永久分配 GPU 图层：很多卡片上永久的 `will-change` 会成倍增加 GPU 内存占用。演示只有一张卡片，所以一直开着。
- **VanillaTilt.js**：一个零依赖的库，处理这种模式，可配置倾斜、眩光、缩放和透视。gzip 后 2KB——生产环境用它而不是手写。
- **`overflow: hidden` 会压平 3D**：任何 `overflow` 不是 `visible` 的祖先（或 `filter`、小于 1 的 `opacity`）都会把 `transform-style` 强制为平面的使用值，于是子元素上的 `translateZ` 会悄悄失效。裁剪自己内容的卡片因此不能同时是 `preserve-3d` 容器——给它自己的 `perspective`，让里面的深度层仍然能投影，或者把裁剪移到 3D 链之外的包装层上。
- **性能**：GPU 合成元素上的 `rotateX`/`rotateY` 以 60fps 运行，不用绘制。跟随指针的其他东西也保持在 `transform` 和 `opacity` 上：演示把光作为一个图层移动（见“光泽高光”），卡片的阴影固定在 CSS 里，随卡片一起转动。每次移动都重写渐变或 `box-shadow` 会每帧重绘整张卡片，手机上看得出来，在网格里还会累加。
- **减弱动态**：演示不会自己倾斜，指针离开时卡片会直接弹回平放而不是缓回去。生产环境里，让这些用户的卡片保持平放。

## 另见
- [3D 模型环绕](../3d-model-orbit/) — 用 WebGL 画出的真正 3D 物体
- [悬停状态动画](../../04-micro-interactions/hover-state/) — 没有 3D 的平面悬停反馈
- [2.5D / 伪 3D](../2-5d-pseudo-3d/) — 不同深度的图层跟着指针移动
- [全息卡片](../holographic-card/) — 同样的倾斜，彩虹箔片和光斑跟随指针
