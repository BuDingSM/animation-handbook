# 景深照片

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/depth-map-photo/)

## 它是什么
景深照片在你指着它时，让一张平面图片看起来有了立体感。图片旁边放着一张同样大小的灰度景深图，近处是白色，远处是黑色；指针移动时，图片的每个部分按它的灰度移动。近处比远处滑得更多，眼睛把这种差别读作景深，就像你晃动脑袋时一样。

## 什么时候用
- 邀请指针来探索的主视觉图片和页眉
- 有清晰前景和背景的旅行、风景和作品集照片
- 专辑封面、海报和书封
- 产品站在纯色背景前的产品照

## 工作原理
两张图片作为纹理传给显卡：照片和它的景深图。演示在 2D 画布上用同样的六个形状自己画出这两张图（天空 0、山 0.2、丘陵 0.45、草地 0.65、树 0.9、地面 0.9 到 1）：一次画成彩色，一次每个形状填上自己的灰度。树和它站着的地面一样近，所以它的根永远不会相对地面滑动。两张图都画得比舞台稍大，这样最大的位移也不会露出边缘。然后景深图里的近处形状向外长出几个像素：十六个副本在环上移动宽度的 1/250（每隔一个移动一半），保留每个像素更亮、更近的值。再用宽度 1/500 的模糊抹平阶梯：

```js
const r = SW / 250;                         // SW: the width the pictures are painted at
b.drawImage(depth, 0, 0);
b.globalCompositeOperation = 'lighten';     // keep the lighter (nearer) value of each pixel
for (let i = 0; i < 16; i++) {
  const d = i % 2 ? r / 2 : r;
  b.drawImage(depth, Math.cos(i * Math.PI / 8) * d, Math.sin(i * Math.PI / 8) * d);
}
b.globalCompositeOperation = 'source-over';
b.filter = `blur(${Math.max(1, Math.round(SW / 500))}px)`;
b.drawImage(b.canvas, 0, 0);
```

片元着色器接着画出舞台的每个像素。深度为 `t` 的一处画面，按指针的偏移乘以 `t − focus` 移动，其中 focus 是保持不动的深度：0.4，刚好在 0.45 的丘陵之前。要画一个像素，着色器从后往前算：从最近到最远尝试 24 个深度，在每次尝试对应的位置查景深图，停在第一个至少和尝试值一样近的位置。于是最近的表面获胜，因为它盖住了后面的东西。再把最后一步对半分五次，就能精确找到边缘，边缘保持锐利而不是阶梯状：

```glsl
float t = 1., tp = 1., h = texture2D(uDep, uv + uOff * (1. - uFocus)).r - 1.;
for (int i = 0; i < 24; i++) { if (h >= 0.) break; tp = t; t -= 1. / 24.; h = texture2D(uDep, uv + uOff * (t - uFocus)).r - t; }
for (int i = 0; i < 5; i++) { float m = (t + tp) * .5; if (texture2D(uDep, uv + uOff * (m - uFocus)).r >= m) t = m; else tp = m; }
vec2 p = uv + uOff * (t - uFocus);
gl_FragColor = vec4(texture2D(uImg, p).rgb, 1.);   // the Shows the depth map switch mixes in the map's gray here
```

近处的形状移开时，会露出一道照片里没有的条带，搜索会用形状边缘的像素拉伸来填补。因为近处的形状在景深图里已经长大了，那些边缘像素是背景，所以条带填的是背景，而不是树的拖影。

视图按流逝的时间滑向指针，所以在任何帧率下移动速度都一样：

```js
const k = 1 - Math.pow(1 - 0.1, dt / FRAME);           // 10% of the gap every 60th of a second
sx += (ax - sx) * k;  sy += (ay - sy) * k;             // a: where the pointer aims, s: where the view is (-1 to 1)
gl.uniform2f(uOff, O * sx / SW, -VY * O * sy / SH);    // O: the depth strength times the stage's width; VY: 0.3
```

Pointer Events 让鼠标、手写笔和手指同样能瞄准视图；鼠标离开或手指抬起会把它送回中间。「演示一下」让视图绕着 8 字形走三秒再回到中间。没有 WebGL 时，演示改为每帧在 2D 画布上画出六个形状，每个按自己的深度移动，和 2.5D 图层移动的方式一样。减弱动态下，图片不跟随指针，「演示一下」先显示一侧的视图 1.2 秒，中间没有移动，再显示正面的视图。

图片也可以获得键盘焦点，方向键代替指针：每按一次方向键，瞄准点就向该方向移动图片宽度的 6%（按住会重复移动），按 Escape 或把焦点移走会把视图送回中间。减弱动态下方向键什么也不移动，和指针一样。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 景深强度 | 中等 | 指针到达图片边缘时，最近的部分相对最远的部分滑动多远：轻微是图片宽度的 2%，中等 3.5%，强烈 5%；上下移动只有约三分之一，这样拉伸的条带比较小 |
| 显示景深图 | 关 | 用灰色景深图代替图片画出来，以同样的方式移动：白色是近处，黑色是远处 |

## 生产注意事项
- **获取景深图**：深度估计模型（MiDaS、Depth Anything）能从任何照片生成一张，手机用人像模式拍照时会保存一张，图片编辑器也可以手工画一张，每层一个灰色形状。保存为与照片同样大小的灰度图。
- **边缘**：近处的物体移动时，会露出照片里没有的背景。保持强度低一些，上下移动的占比比左右小一些，在景深图里把近处的形状长大几个像素并稍微模糊，在物体接触地面的地方给它一个统一的深度；轮廓干净的手工地图最有帮助。
- **纹理**：用线性过滤和夹到边缘的包裹方式，这样 WebGL 1 不用 mipmap 就能接受任何大小的图片。上传时翻转图片（`UNPACK_FLIP_Y_WEBGL`），图片要与页面同源或带 CORS 头，否则 WebGL 拒绝读取。
- **手机倾斜**：手机自身的倾斜（`deviceorientation`）可以代替手指驱动位移；iOS 会先向用户请求许可。
- **性能**：着色器每个像素大约读取二十次景深图，对显卡来说很便宜。演示把像素密度封顶在 2（手机上 1.5），只在视图移动时绘制。
- **上下文丢失**：GPU 重置或恢复的标签页会触发 `webglcontextlost`；在上面调用 `preventDefault()`，并在 `webglcontextrestored` 里重建着色器、重新上传纹理。
- **库**：Three.js 用 `ShaderMaterial` 运行同样的着色器，或按景深图把网格推出去；PixiJS 有 `DisplacementFilter`；Facebook 的 3D 照片把景深图变成网格，并补上被遮住的背景。

## 另见
- [2.5D / 伪 3D](../2-5d-pseudo-3d/) — 按不同幅度滑动的平面图层
- [悬停图片扭曲](../image-distortion-hover/) — 在指针周围移动像素的着色器
- [视差 3D 倾斜](../parallax-3d-tilt/) — 一张卡片朝指针倾斜
- [3D 轮播](../carousel-3d/) — 围成一圈、在 3D 中转动的卡片
