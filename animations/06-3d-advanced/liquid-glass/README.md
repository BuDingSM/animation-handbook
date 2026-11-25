# 液态玻璃

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/liquid-glass/)

## 它是什么
液态玻璃让一块面板看起来像躺在页面上的一块厚厚的透明玻璃。靠近圆角边缘的地方，身后的画面会弯曲放大，就像透镜弯曲的边缘弯折光线，中间几乎保持透明，边缘上还有一条细细的高光。在 Chromium 浏览器里，弯曲是施加在面板身后东西上的位移滤镜；其他浏览器把同一块面板显示为磨砂玻璃。

## 什么时候用
- 浮在照片或滚动内容上的悬浮菜单和标签栏
- 放在主视觉图片或视频上的媒体控件和小卡片
- 可以拖过地图、图表或产品照片的放大镜
- 产品页或作品集上一个触感十足、引人注目的控件

## 工作原理
**滤镜。**SVG 滤镜按第二张图片（位移图）读取的量移动每个像素：图的红色通道是横向位移，绿色通道是上下位移，128 表示没有位移。从 `backdrop-filter` 引用它，滤镜弯曲的是面板身后的东西，而不是面板自己的内容：

```html
<svg width="0" height="0" style="position:absolute">
  <filter id="lg-filter" filterUnits="userSpaceOnUse" x="0" y="0" width="280" height="174" color-interpolation-filters="sRGB">
    <feImage href="data:image/png;base64,…" x="0" y="0" width="280" height="174" preserveAspectRatio="none" result="map"/>
    <feDisplacementMap in="SourceGraphic" in2="map" scale="59" xChannelSelector="R" yChannelSelector="G"/>
  </filter>
</svg>
```

```css
.lg-refract .lg-glass { backdrop-filter: url(#lg-filter) blur(var(--lg-blur)) saturate(1.3); }
```

滤镜区域和位移图都是面板自己的像素尺寸，所以图与面板对齐，面板尺寸变化时脚本会同时设置两者。`color-interpolation-filters="sRGB"` 让 128 保持在中间：默认的线性色彩空间里滤镜会把那个灰色读成约 0.22，让一切都位移。

**位移图。**演示在画布上画出位移图，面板每个 CSS 像素对应一个像素。对每个像素，它找出 `e`（离圆角边缘的距离）和 `(nx, ny)`（向外的方向）。在边缘内侧（面板半高的 70%），像素被告知从更靠里的地方读取画面，越靠近边缘越靠里，所以边缘放大了身后的东西；处处向中间的一点拉力让整块面板也稍微放大一点。圆角之外的位移图保持在 128：

```js
const m = e < 0 ? 0 : (1 - Math.min(1, e / B)) ** 2, z = e < 0 ? 0 : MAG;   // B: the rim's width, MAG = 0.15
d[k]     = 128 - 127 * (nx * m + z * X / hw) / (1 + MAG);                  // red: sideways
d[k + 1] = 128 - 127 * (ny * m + z * Y / hh) / (1 + MAG);                  // green: up and down
```

滤镜最多把一个像素移动它 `scale` 的一半，所以「弯曲程度」只改变 `scale`（弯曲的两倍乘以边缘宽度，再乘以 1.15 的向中间拉力）；位移图只在面板改变尺寸或形状时重画。

**哪些浏览器会弯曲。**Chrome、Edge 和 Opera 等基于 Chromium 的浏览器会在 `backdrop-filter` 里画 SVG 滤镜（演示在 Chrome 和 Edge 里检查过）。Safari 和 Firefox 不画，特性检测也测不出来：`CSS.supports()` 只检查规则是否有效，不检查弯曲是否出现。演示把弯曲交给在 `navigator.userAgentData` 里把 Chromium 列为品牌的浏览器——只有基于 Chromium 的浏览器才有——其他浏览器一律显示磨砂玻璃：背景的模糊配上同样的着色、边缘和阴影，边缘里还有一道随「弯曲程度」变强的光晕。那里舞台上方的帮助行会加上“这里是磨砂玻璃：弯曲效果只在电脑或安卓上的 Chrome 或 Edge 里显示。”iPhone 和 iPad 上所有浏览器，包括 Chrome 和 Edge，都是基于 Safari 的引擎，显示的都是磨砂玻璃。`navigator.userAgentData` 只在安全页面（https 或 localhost）上存在，所以通过普通 http 在网络地址上打开页面的 Chrome 得到的也是磨砂玻璃。

```js
const REFRACT = !!(navigator.userAgentData && navigator.userAgentData.brands.some(b => b.brand === 'Chromium'))
  && CSS.supports('backdrop-filter', 'url(#lg-filter)');
stage.classList.toggle('lg-refract', REFRACT);
```

**液体的拉伸。**面板只靠 `transform` 移动。它的速度从一个绘制帧到下一个测量，再经过几帧平滑，决定它沿运动方向应该拉伸多远：每秒走过一个面板宽度约 6%，最多 12%。一个按 1/60 秒固定步长走的弹簧跟随这个目标，所以拉伸会滞后一点，面板停下时会冲过头、短暂地晃动。拉伸保持面积：沿运动方向变长，横向变短，作为围绕面板中心的一个矩阵：

```js
const ta = e * (ux * ux - uy * uy), tb = e * 2 * ux * uy;          // the target for the motion's direction (ux, uy)
for (acc += dt * f; acc >= FRAME; acc -= FRAME) {                     // f is 1/3 in slow motion
  va += -.08 * (a - ta) - .1 * va; vb += -.08 * (b - tb) - .1 * vb; a += va; b += vb;
}
glass.style.transform = `translate(${x - gw / 2}px, ${y - gh / 2}px) matrix(${1 + a}, ${b}, ${b}, ${1 - a}, 0, 0)`;
```

「演示一下」沿三段滑行（到左端、横穿到右端、再回到出发的地方），每段按长度缓入缓出，总共 3.6 秒。舞台尺寸变化时（调整大小，或手机横过来），玻璃、滑行的终点、每次运行的每个点和一次按住的「演示一下」要回去的地方都保持它们在舞台上的占比；原来在静止位置的会去新的静止位置。方向键一次移动一个目标（舞台较短边的 8%，按住 Shift 是三步），面板按每六十分之一秒弥合一段差距滑向它，在任何屏幕上都一样。

**减弱动态。**面板在拖动时仍然跟随指针或手指，但绝不拉伸、晃动或滑行：方向键、轻触和「重置」会立刻移动它，「演示一下」把它放到右端 1.2 秒再回来。减弱动态下的每一帧都把拉伸和它的速度设为零，玻璃移动时打开减弱动态会让一次运行停在原地、一次滑行在目标处结束，所以什么也不会留下来拉伸或移动。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 弯曲程度 | 中等 | 边缘最深处的位移，在最边缘，作为边缘宽度（面板高度的 35%）的占比：轻微 25%，中等 42%，强烈 60%，再加向中间的一点拉力，让整块面板稍微放大，额外增加 15%。超过一半时，边缘会折成一条细细的镜像带，就像厚玻璃的边缘那样。其他浏览器则在边缘里显示一道淡淡的白光，分别模糊 12、23 或 36px |
| 磨砂程度 | 透明 | 背景模糊多少：透明不模糊，轻微 1.5px，磨砂 4px（在玻璃会弯曲的地方）；其他浏览器的磨砂玻璃是 3、8 和 16px |
| 着色 | 白色 | 玻璃加上的颜色，白色按 9% 混合，其他颜色按 20% 混合 |
| 形状 | 卡片 | 卡片：宽是高的 1.6 倍，圆角是高度的三分之一；胶囊：宽是高的 2.7 倍，两端是圆的；圆形：圆形的放大镜。卡片是舞台宽度的 30%，从 150 到 280px（胶囊宽 1.12 倍，圆形宽 0.68 倍），任何形状都不高于舞台的 55% |
| 显示弯曲图 | 关 | 把位移图铺在玻璃上：红色是横向位移，绿色是上下位移，灰色是没有位移 |

## 生产注意事项
- **目前只有 Chromium**：`backdrop-filter: url()` 在 Chromium 里弯曲背景；Safari 和 Firefox 需要回退方案，所以两种浏览器里都要检查页面。覆盖更多浏览器的办法是把背景的副本放进面板，再用 `filter: url()` 对那个副本应用同样的滤镜，普通元素上也能用；只在你能控制面板身后是什么的地方才行。
- **检测**：`CSS.supports()` 只检查语法，所以像演示那样再加一个 Chromium 检查，并把回退方案做得足够好，能当作整个效果。
- **成本**：面板移动的每一帧，浏览器都会重新过滤背景，工作量随面板面积增长。玻璃只用在菜单、控件和小卡片上，不要用在全屏的薄片上。位移是每像素一次纹理查找；模糊才是贵的部分，所以磨砂保持轻微。演示的面板是舞台的一个占比，所以手机上更小。
- **让位移图跟上面板走**：为另一个尺寸画的位移图会把弯曲移离边缘。调整大小和改变形状时重画；只改变 `scale` 很便宜。
- **玻璃上的文字**：折射的、移动的颜色上的文字可能掉到 4.5:1 以下。给它一层深色半透明层或实心底衬，像动态毛玻璃那样。
- **库**：滤镜不需要库。拖动和晃动可以用 GSAP 的 Draggable 加弹性缓动，或 Framer Motion 带弹簧的拖动；几个开源 React 和 Vue 组件包装了同样的 SVG 位移技术。原生应用里，苹果的 Liquid Glass（iOS 和 macOS 26）是一种系统材质。

## 另见
- [动态毛玻璃](../glassmorphism-animated/) — 磨砂玻璃，模糊身后的东西而不是弯曲它
- [悬停图片扭曲](../image-distortion-hover/) — 用着色器画出的、在指针周围弯曲的图片
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜，同时一道光滑过
