# 色差

[在线演示](https://animation.xizanghuapi.cn/animations/06-3d-advanced/chromatic-aberration/)

## 它是什么
色差模仿廉价镜头的缺陷：红、绿、蓝三色光落在略微不同的位置，在边缘留下彩色边纹。演示把一个词的红、绿、蓝三个副本叠在一起，重叠处加起来是白色，再把红和蓝的副本错开。裂开的幅度可以一呼一吸，也可以故障式跳动。

## 什么时候用
- 音乐、游戏和前沿科技产品的故障美学品牌
- 做旧印刷或复古 VHS 视觉风格
- 应该让人感觉物理上“不对劲”的错误或警告状态
- 通过光学畸变表明可以交互的悬停效果

## 工作原理
三个完全相同的文字或图片元素叠在一起。每个用 `filter` 或明确的颜色染成单一颜色通道，再以 `mix-blend-mode: screen` 混合，让每个通道只贡献自己的颜色。红和蓝两层从绿色（中间）层向相反方向偏移：

```css
.ca-wrap { position: relative; display: inline-block; }

.ca-layer      { position: absolute; inset: 0; mix-blend-mode: screen; }
.ca-layer.base { position: relative; mix-blend-mode: normal; }  /* sizing */

.ca-r .text { color: #ff2020; }   /* red channel  */
.ca-g .text { color: #20ff20; }   /* green channel (center, no offset) */
.ca-b .text { color: #2060ff; }   /* blue channel  */
```

```js
function applyOffset(amount, angleDeg) {
  const rad = angleDeg * Math.PI / 180;
  const dx = (Math.cos(rad) * amount).toFixed(1);
  const dy = (Math.sin(rad) * amount).toFixed(1);
  rLayer.style.transform = `translate(${dx}px, ${dy}px)`;
  bLayer.style.transform = `translate(${-dx}px, ${-dy}px)`;  // opposite
}
```

**动态故障**——周期性地跳到很大的随机偏移，再跳回来：

```css
@keyframes ca-glitch-r {
  0%, 80%, 100% { transform: translate(4px, 0); }
  82%           { transform: translate(12px, -4px); }
  84%           { transform: translate(4px, 0); }
  86%           { transform: translate(-8px, 4px); }
}
```

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 运动方式 | 脉冲 | 脉冲每 2 秒把裂开幅度放大到 1.8 倍再收回来；故障让红和蓝的副本在每 4 秒快结束时跳动两次，每次 80ms，无论「方向」如何都始终朝同一个方向：红色先向右跳 3 倍裂开距离、向上跳 1 倍，再向左跳 2 倍、向下跳 1 倍，蓝色朝相反方向；无保持静止 |
| 颜色分离 | 中等 | 红和蓝的副本离原词多远：小是 2px，中等 4px，大是 8px；超过约 6px，词就难读了 |
| 方向 | 斜向 | 横向把颜色左右分开；斜向以 30° 分开；上下把颜色垂直分开 |
| 柔和的彩色边缘 | 关 | 把红和蓝的副本模糊 2px，看起来更像真正的镜头，而不是数字位移 |
| 你的文字 | PRISM | 会裂开的词 |

## 生产注意事项
- **`mix-blend-mode: screen`** 需要深色背景——screen 混合会把 RGB 值相加，所以在白色背景上三个通道加起来是白色，偏移看不见。这个效果只在深色背景上有效。
- **WebGL 着色器做法**：要做径向色差（边缘更强，符合真实镜头的表现），片元着色器按每个像素在三个不同的 UV 偏移处采样纹理。CSS 做法只支持均匀的方向偏移。
- **文字可读性**：强度超过约 6px 时文字就难读了。高强度只用于短标题或装饰元素，绝不用在正文上。
- **性能**：带 `mix-blend-mode` 的 CSS 层叠元素会在 GPU 上触发合成。三个 `screen` 混合的层很便宜。手机上不要把它用在大面积图片上。
- **库**：图片和 3D 场景可以用 three.js 的 postprocessing 库里的 `ChromaticAberrationEffect`（可选径向衰减），PixiJS 有 `RGBSplitFilter`。

## 另见
- [故障文字](../../05-text-typography/glitch-text/) — 红青色条带撕过一个词
- [WebGL 着色器动画](../webgl-shader-animation/) — 可以逐像素分离颜色的着色器
- [乱码/故障文字](../../05-text-typography/scramble-text/) — 随机符号逐个锁定为真实文字
