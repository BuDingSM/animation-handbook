# 星级评分

[在线演示](https://animation.xizanghuapi.cn/animations/04-micro-interactions/star-rating/)

## 它是什么
星级评分让人通过选一颗星来打分。指针沿着这排星星移动时，星星会填到它下面的那颗，这样选择在做出之前就能看到；点击一颗星会选中它，并让它弹出。方向键也能改变评分；一个设置让每颗星可以按半颗算。

## 什么时候用
- 产品、评价和反馈表单，粗略的 1–5 分就够了
- 购买后或支持后的调查（“我们做得怎么样？”）
- 卡片或详情视图里内联显示的内容或媒体评分
- 任何数字输入会比快速点按显得更重的地方
- 不适合精确测量——五（或十）个档就是全部意义；需要更细的值时，用滑块或数字字段

## 工作原理
每颗星是一个用 `currentColor` 画的星形，上面叠着一个用强调色画的第二份复制品，裁剪到左边的 50%。一颗满星只是形状上的颜色互换，从柔和到强调；半颗星露出裁剪过的强调色复制品。因为互换是 `color` 上的 `transition`，指针扫过这排星星时，星星会一颗接一颗填满——动画就是这个级联，而不是每颗星的宽度补间。确认会重新触发选中星星上的 `transform: scale` 弹出：

```css
.star{ color: var(--ui-muted); transition: color var(--fill-dur) ease; }
.star.full{ color: var(--ui-accent); }
.star .half{ position:absolute; width:50%; overflow:hidden; color:var(--ui-accent); opacity:0; }
.star.half-on .half{ opacity:1; }

@keyframes pop{ 0%{transform:scale(1)} 35%{transform:scale(var(--pop))} 70%{transform:scale(.94)} 100%{transform:scale(1)} }
.star.pop{ animation:pop .42s cubic-bezier(.22,1,.36,1); }
```

星星里的指针位置决定数值——左半边四舍五入到 `.5`，右半边到整数——`pointermove`/`pointerleave` 会预览它而不确认。弹出通过去掉再重新加上 `pop` 类来重新触发（`star.classList.remove('pop'); void star.offsetWidth; star.classList.add('pop')`），这样重复确认总会有动画。

## 关键参数
| 参数 | 默认值 | 效果 |
|-----------|---------|--------|
| 弹出大小 | 中等 | 选中的星星弹出时变大多少：1.15、1.35 或 1.6 倍；超过大约 1.6 会显得像橡胶 |
| 速度 | 普通 | 每颗星星填满颜色要花多长时间：慢是 300ms，普通 180ms，快是 110ms；它决定填充跟随指针的平滑程度 |
| 允许半星 | 关 | 指向星星的左半边给半分，方向键按半分移动 |
| 星星数量 | 五个 | 三个、五个或十个星星；五个是常用的刻度 |

## 生产注意事项
- **无障碍**：这个演示把这排暴露为一个 `role="slider"`，带 `aria-valuemin/max/now` 和一个 `aria-valuetext`（"3 / 5"），从键盘处理半步很干净。一个 `role="radiogroup"` 加 `radio` 星星是另一种地道选择，更适合离散的整星评分；按有没有半值来选模型。
- **键盘**：方向键按当前增量步进，Home 清到 0，End 跳到最大。控件必须能聚焦（`tabindex="0"`），并显示可见的焦点环——只用鼠标评分是不够的。
- **不要只依赖颜色**：填充由这排下面的数字读数加强，这样色盲用户和灰度下也能看到数值。永远不要只用色相编码分数。
- **半星处理**：按悬停星星里指针的横向位置确定半颗，并把确认值和悬停预览分开，这样离开这排时会恢复确认的评分，而不是一个 stray 的悬停状态。
- **触摸目标**：每个星星框至少 44×44px，控件用指针事件，所以悬停预览和点按确认在触摸上都好用。没有 CSS `:hover` 样式，所以点按后什么都不会留在画上；预览来自 `pointermove`，手指的拖动会问 `document.elementFromPoint` 它下面是哪颗星。
- **库的等价物**：Framer Motion 可以用 `whileTap`/`animate` 缩放弹簧驱动弹出；很多表单套件（比如无头 UI 库里的评分输入）自带单选组语义，你只需要给星星做样式。

## 另见
- [点赞爱心迸发](../heart-burst/) — 点赞按钮迸出小爱心
- [按钮按压缩放](../button-press-scale/) — 按下时缩小的按钮
- [开关滑动](../toggle-switch/) — 另一个让状态有动画的小控件
