# 2.5D / 伪 3D

[在线演示](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/2-5d-pseudo-3d/)

## 它是什么
2.5D 用平面图层假装出景深。画面按远近分成图层，镜头移动时，近处的图层比远处滑得更多，就像从火车车窗看出去一样。老卡通和电子游戏就是用同样的手法做出纵深感的。

## 什么时候用
- 带插画场景的主视觉区，想让它显得有纵深
- 作品集页眉和落地页环境
- 游戏界面叠加层，纵深感能强化游戏世界
- 交互地图或环境，镜头移动能揭示景深

## 工作原理
每个图层都有一个 CSS `transform: translate(x, y)`，让它朝着与指针相反的方向滑动：x = `-normX * strength * depth`，y = `-normY * strength * depth * 0.4`，其中 `normX` 和 `normY` 是指针在舞台上的位置，从 -0.5 到 0.5。深度为 0 的图层不动，深度为 1 的图层移动幅度最大：

```js
const LAYERS = [
  { el: document.getElementById('sky'),    depth: 0.05 },
  { el: document.getElementById('mtns'),   depth: 0.2  },
  { el: document.getElementById('hills'),  depth: 0.4  },
  { el: document.getElementById('trees'),  depth: 0.7  },
  { el: document.getElementById('ground'), depth: 1.0  },
];

stage.addEventListener('pointermove', e => {
  const r = stage.getBoundingClientRect();
  const normX = (e.clientX - r.left) / r.width  - 0.5;  // -0.5 to 0.5
  const normY = (e.clientY - r.top)  / r.height - 0.5;
  LAYERS.forEach(({ el, depth }) => {
    el.style.transform = `translate(
      ${-normX * STRENGTH * depth}px,
      ${-normY * STRENGTH * depth * 0.4}px
    )`;
  });
});
```

演示有七个图层，从天空（深度 0.05）到地面（深度 1.0）。镜头停在指针离开的位置。它还会监听 `pointerdown`，所以轻触能移动镜头，手指横向拖动也能带着镜头走，和鼠标一样。

按下「演示一下」时，演示会用一段三秒的扫动代替指针驱动同样的图层，最后停在中间：

```js
function sweep(now) {                       // start = performance.now() when Show me is pressed
  const p = Math.min(1, (now - start) / 3000);
  const normX = 0.45 * Math.sin(2 * Math.PI * p);
  const normY = 0.2 * Math.sin(Math.PI * p);
  LAYERS.forEach(({ el, depth }) => {
    el.style.transform = `translate(${-normX * STRENGTH * depth}px, ${-normY * STRENGTH * depth * 0.4}px)`;
  });
  if (p < 1) requestAnimationFrame(sweep);
}
```

演示还会融合镜头的起始位置，所以第二次按下，或者一次被真实指针打断的运行，会从镜头所在的位置继续，而不是跳回去。「重置」会把镜头放回中间。

场景也可以获得键盘焦点，方向键代替指针：键盘焦点会显示镜头所在的圆环，每按一次方向键，镜头就向该方向移动舞台宽度的 6%（按住会重复移动）。按 Escape 或把焦点移走会隐藏圆环，镜头停在原地，和指针离开时一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 镜头移动幅度 | 中等 | 指针从场景一边移到另一边时，最近的图层走多远：小是 20px，中等 40px，大是 70px（从中间算起每边一半），上下方向只有 40%；最远的图层只走其中的 5% |
| 图层标签 | 关 | 给每个图层标上它跟随镜头移动的比例，从天空的 5% 到地面的 100% |

## 生产注意事项
- **每个图层都必须比舞台宽**：镜头平移时，图层必须伸出可视范围之外，否则会露出空白边缘。每个图层多留 10–20% 的横向溢出。
- **CSS `perspective` 与 `translateZ`**：CSS 3D 透视用更少的 JavaScript 就能得到同样的结果——在容器上放 `perspective: 800px`，每个图层用 `translateZ(depth)`。再用鼠标旋转整个容器，一个变换就能做出视差效果。
- **滚动驱动的变体**：把 `pointermove` 换成 `scroll`，把滚动位置映射到镜头偏移。再结合吸顶定位，在用户滚动经过时把场景固定住。
- **游戏引擎**：这是 2D 游戏引擎（Phaser、PixiJS、Godot）的原生手法，“视差滚动”是其中一等功能。每个图层指定一个滚动系数（0 到 1）。
- **优化**：伸出视口之外的图层会触发绘制。在父元素上用 `overflow: hidden`，每个图层用 `transform: translateZ(0)`，把它们提升为 GPU 合成层。

## 另见
- [视差景深](../../01-scroll-based/parallax-depth-of-field/) — 同样的分层景深，由滚动驱动
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜
- [滚动驱动 3D 旋转](../scroll-driven-3d-rotation/) — 滚动移动一个 3D 物体
- [景深照片](../depth-map-photo/) — 一张照片，近处比远处移动得更多
