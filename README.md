<div align="center">

<a href="https://matinmonshizadeh.github.io/animation-handbook/"><picture><source media="(prefers-reduced-motion: reduce) and (max-width: 600px)" srcset="docs/hero-phone-still.webp"><source media="(prefers-reduced-motion: reduce)" srcset="docs/hero-still.webp"><source media="(max-width: 600px)" srcset="docs/hero-phone.webp"><img src="docs/hero.webp" alt="手册里的六个动画同时播放：文字粒子、水面涟漪、合成波网格、液态玻璃、火焰效果和前后对比滑块，是 176 个网页动画效果中的六个" width="820"></picture></a>

# 动画手册

**176 个网页动画效果的可视化参考：每个动画都能在线播放，大多数可以用简单的设置调整，还附带可直接复制的提示词，粘贴到 AI 助手里就能用。免费使用，不用注册。**

[**打开在线手册 →**](https://matinmonshizadeh.github.io/animation-handbook/)

[![Live site](https://img.shields.io/website?url=https%3A%2F%2Fmatinmonshizadeh.github.io%2Fanimation-handbook%2F&label=live%20demo&up_message=online&color=91bdff)](https://matinmonshizadeh.github.io/animation-handbook/) [![Stars](https://img.shields.io/github/stars/matinmonshizadeh/animation-handbook?style=flat&color=79de9d)](https://github.com/matinmonshizadeh/animation-handbook) [![License](https://img.shields.io/github/license/matinmonshizadeh/animation-handbook?color=c6a1ff)](LICENSE) [![Techniques](https://img.shields.io/badge/techniques-176-ff9d5c)](#full-catalog) [![Dependencies](https://img.shields.io/badge/dependencies-0-ff94a9)](#设计原则) [![Build](https://img.shields.io/badge/build-none-62ddcf)](#设计原则)

</div>

---

## 这是什么

这里收录了 176 个网页动画效果，全部放在一处，每个都能实际看到效果。每个页面都会在线播放它的动画，并给出一句现成的提示词，粘贴到 AI 助手里，不用自己写代码就能把这个动画加到你的网站上。大多数页面还可以用几个简单的设置调整动画，复制的提示词会列出你的选择。每个文件夹里的 `README.md` 会讲这个效果的原理、关键参数和生产中的注意事项，适合想亲手实现的人看。

全部都是普通的 HTML、CSS 和 JavaScript：没有构建步骤，没有框架，没有 CDN。从仓库的副本里直接打开任意页面的 `index.html`，演示就能离线运行；[本地运行](#本地运行）讲的是哪些部分需要本地服务器。做这个手册是为了*启发*和*教学*，不是为了卖一个库。

## 怎么用

1. **选一个动画。**在[首页](https://matinmonshizadeh.github.io/animation-handbook/)上用自己的话描述它，比如「点击时会弹跳的按钮」，或者选它在页面上的位置：按钮、文字、图片与卡片、背景、菜单与表单、加载与提示、页面开场、滚动或页面切换。在某个网站上看到一个动画却不知道叫什么？在[查找器](https://matinmonshizadeh.github.io/animation-handbook/?view=find)里回答三个简短的问题：什么在动、什么时候动、怎么动。
2. **看它动起来。**每个动画都能在线播放。打开它的页面可以放大看，还能试几个简单的设置，比如速度或方向，这样你能确切看到会得到什么效果。演示上方，**看内部**会展示构成这个效果的部件（参考线、隐藏图层、标出来的数字），**对比**会把它和一个去掉关键想法的版本并排展示。
3. **复制提示词。**粘贴到你的 AI 助手里。它用简单的话描述这个动画，并带上你的设置。首页上的每张卡片也有一个「复制提示词」按钮。

<div align="center">
<a href="https://matinmonshizadeh.github.io/animation-handbook/"><picture><source media="(prefers-reduced-motion: reduce) and (max-width: 600px)" srcset="docs/demo-phone-still.webp"><source media="(prefers-reduced-motion: reduce)" srcset="docs/demo-still.webp"><source media="(max-width: 600px)" srcset="docs/demo-phone.webp"><img src="docs/demo.webp" alt="手册的录屏：在首页点「按钮」，会出现带实时预览的卡片；打开「成功彩带」播放它的演示；点「复制提示词」会显示「已复制」" width="820"></picture></a>
</div>

## 知名效果

在知名网站上看到喜欢的效果？[知名效果](https://matinmonshizadeh.github.io/animation-handbook/famous/)收集了人们在这些网站上欣赏的效果。每个效果先展示我们自己做的可用版本，用简单的话讲你看到的是什么，指出手册里哪些动画能做出类似的效果，并给你一句提示词，让你自己动手做。手册与这些网站及其所有者没有任何关联。

## 分类

| # | 分类 | 数量 |
|:--:|----------|:-----:|
| 01 | [滚动驱动](#01) | 27 |
| 02 | [入场与退场](#02) | 15 |
| 03 | [页面切换](#03) | 17 |
| 04 | [微交互](#04) | 44 |
| 05 | [文字与排版](#05) | 19 |
| 06 | [3D 与高级效果](#06) | 28 |
| 07 | [氛围与背景](#07) | 26 |

## 完整目录

全部 176 个动画，每个都链接到在线演示。

每个动画的代码和 README 都在 [animations/](animations/) 下对应的文件夹里。

<a id="01"></a>

### 01 · 滚动驱动 · 27 个动画

<sub>由滚动位置驱动的动画——视差、吸顶、擦除、吸附和滚动叙事。</sub>

<details><summary>浏览「滚动驱动」的全部 27 个动画</summary>

- **[视差景深](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/parallax-depth-of-field/)** — 各图层随焦点变化以各自的速度移动。最适合电影感开场。
- **[视差滚动](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/parallax-scrolling/)** — 远处的图层比近处的移动得更慢。最适合主视觉场景。
- **[反向滚动列](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/reverse-scrolling-columns/)** — 两侧的列与中间一列朝相反方向滚动。最适合作品集。
- **[封面卡片变固定顶栏](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/cover-card-to-fixed-header/)** — 高封面滚动时收成细顶栏。最适合文章页。
- **[联系人列表飞入飞出](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/fly-in-fly-out-contact-list/)** — 行在靠近顶部或底部边缘时淡入淡出、滑入滑出。最适合长列表。
- **[层叠卡片](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/stacking-cards/)** — 卡片吸顶并叠成一摞。最适合分步讲述。
- **[滚动触发动画](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-trigger/)** — 动画在设定的滚动点开始、跟随并固定。最适合落地页。
- **[滚动 scrub 动画](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scrub-animation/)** — 滚动像拖视频进度条一样正放倒放。最适合产品演示。
- **[固定动画](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/pin-animation/)** — 一部分固定不动，文字随滚动变化。最适合功能列表。
- **[滚动吸附](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/snap-scrolling/)** — 盒子一次停在一个完整区域上。最适合幻灯片和画廊。
- **[滚动叙事](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scrollytelling/)** — 旁边的图片随你往下读而变化。最适合数据故事。
- **[滚动显现](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/reveal-on-scroll/)** — 卡片越过盒子里的一条线时出现。最适合长落地页。
- **[交错显现](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/stagger-reveal/)** — 一组里的元素随滚动依次出现。最适合卡片网格。
- **[横向滚动](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/horizontal-scroll/)** — 往下滚，一行面板横向滑过。最适合作品集。
- **[吸顶区域](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/sticky-section/)** — 一个区域固定不动，内容随之变化。最适合功能导览。
- **[数字滚动计数](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/counter-animation/)** — 数字滚动进入视野时开始累加。最适合统计数据。
- **[阅读进度条](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/progress-bar/)** — 往下读，进度条随之填满。最适合长文章。
- **[区域擦除切换](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/section-wipe/)** — 每个区域从上一个上面滑过。最适合全屏故事。
- **[图片缩放进入](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/zoom-into-image/)** — 小窗口随滚动放大填满整个盒子。最适合主视觉图片。
- **[滚动图片序列](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-image-sequence/)** — 滚动像翻书一样播放一串图片。最适合产品展示。
- **[平滑（惯性）滚动](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/smooth-scroll/)** — 滚动滑到停下，而不是一下跳到位。最适合作品集网站。
- **[滚动文字填充](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/text-fill-on-scroll/)** — 词语随滚动一个接一个亮起来。最适合关键语句。
- **[滚动速度倾斜](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-velocity-skew/)** — 滚得快时行会倾斜，停下时回正。最适合画廊。
- **[SVG 线条滚动描绘](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/svg-line-draw/)** — 线条随滚动沿路线自己画出来。最适合时间线。
- **[滚动侦测导航](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scrollspy-nav/)** — 菜单高亮你正在读的章节。最适合长文档。
- **[滚动驱动背景色](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/scroll-background-color/)** — 背景色随滚动经过的区域而变化。最适合故事讲述。
- **[滚动隐藏顶栏](https://matinmonshizadeh.github.io/animation-handbook/animations/01-scroll-based/hide-on-scroll-header/)** — 往下滚隐藏顶栏，往上滚显示出来。最适合移动端页面。

</details>

<a id="02"></a>

### 02 · 入场与退场 · 15 个动画

<sub>元素级的入场与退场动画——淡入、滑动、揭示、翻转和文字依次出现。</sub>

<details><summary>浏览「入场与退场」的全部 15 个动画</summary>

- **[淡入淡出](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/fade-in-out/)** — 淡入出现，淡出离开。适合弹窗和提示框。
- **[滑入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/slide-in/)** — 从一边滑入到位。适合侧边面板和通知。
- **[上滑揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/slide-up-reveal/)** — 文字从一条看不见的边缘后方升起进入视野。适合标题。
- **[缩放进入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/scale-in/)** — 从小放大到完整尺寸。适合弹窗和菜单。
- **[剪裁路径揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/clip-path-reveal/)** — 一个不断变大的形状揭开它，自身保持静止。适合图片和横幅。
- **[幕布揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/curtain-reveal/)** — 一块彩色面板先盖住它，随后滑走。适合开场和 Logo。
- **[拆字揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/split-text-reveal/)** — 文字拆成碎片，依次出现。适合标题。
- **[逐字母出现](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/letter-by-letter-stagger/)** — 一句话逐字母出现，或像打字机一样。适合短标题。
- **[逐词揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/word-by-word-reveal/)** — 词语按阅读节奏依次出现。适合引言和开场。
- **[模糊渐入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/blur-in/)** — 像相机对焦一样从模糊中变得清晰。适合精选卡片和照片。
- **[翻转进入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/flip-in/)** — 像卡片翻面一样以 3D 方式翻入视野。适合卡片和瓷贴。
- **[弹跳进入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/bounce-in/)** — 带着弹簧般的弹跳落定。适合徽章和成功提示。
- **[旋转进入](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/rotate-in/)** — 边旋转边放大进入到位。适合图标、星星和徽章。
- **[碎裂效果](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/shatter-effect/)** — 碎成碎片飞散并淡出。适合画廊和醒目的退场。
- **[墨水扩散揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/02-entrance-and-exit/ink-spread-reveal/)** — 墨滴扩散、汇合，直至画面显现。适合艺术和开场。

</details>

<a id="03"></a>

### 03 · 页面切换 · 17 个动画

<sub>路由或视图之间的整页切换——交叉淡入淡出、滑动、传送门、变形和 View Transitions API。</sub>

<details><summary>浏览「页面切换」的全部 17 个动画</summary>

- **[View Transitions API](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/view-transitions-api/)** — 浏览器帮你完成两个页面之间的切换动画。最适合 Web 应用。
- **[共享元素过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/shared-element-transition/)** — 图片从列表放大，进入下一页的页眉。最适合图库。
- **[变形过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/morph-transition/)** — Logo 随你访问的每个页面变换形状。最适合品牌标志。
- **[交叉淡入淡出过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/crossfade/)** — 旧页面淡出，新页面同时淡入。最适合平静的页面切换。
- **[滑动过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/slide-transition/)** — 页面横向滑入，返回时反向滑出。最适合分步流程。
- **[缩放过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/zoom-transition/)** — 页面切换时缩放，仿佛在纵深中移动。最适合打开详情。
- **[闪光 / 漏光过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/flash-transition/)** — 一道闪光遮住页面切换的瞬间。最适合大胆活泼的网站。
- **[模糊过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/blur-transition/)** — 旧页面模糊淡出，新页面逐渐清晰。最适合摄影网站。
- **[弹性过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/elastic-transition/)** — 新页面滑入时冲过头再弹回来。最适合趣味应用。
- **[传送门 / 隧道缩放](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/portal-zoom/)** — 下一页从你点击的圆圈中展开。最适合重大揭示。
- **[溶解过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/dissolve/)** — 页面碎成小块，让位给下一页。最适合图片画廊。
- **[FLIP 技巧](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/flip-technique/)** — 布局变化时卡片滑到新位置。最适合排序列表。
- **[圆形揭示](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/circle-reveal/)** — 页面从你点击的任意位置展开，收回时缩回该处。最适合应用导航。
- **[覆盖擦除](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/overlay-wipe/)** — 一块彩色面板扫过页面完成切换。最适合作品集网站。
- **[翻页卷角](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/page-curl/)** — 页面像纸一样从角落卷起。最适合数字杂志。
- **[立方体过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/cube-transition/)** — 页面贴在立方体上，旋转展示下一页。最适合图片故事。
- **[像素化过渡](https://matinmonshizadeh.github.io/animation-handbook/animations/03-page-transitions/pixelate-transition/)** — 页面碎成大像素，再清晰成下一页。最适合画廊。

</details>

<a id="04"></a>

### 04 · 微交互 · 44 个动画

<sub>短暂的、由用户触发的动画——悬停、点击、聚焦、加载状态和界面反馈模式。</sub>

<details><summary>浏览「微交互」的全部 44 个动画</summary>

- **[悬停状态动画](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/hover-state/)** — 指向卡片时，它有六种反应方式。适合用在按钮和卡片上。
- **[点击涟漪](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/click-ripple/)** — 按下的位置会扩散出一圈涟漪。适合用在按钮和列表项上。
- **[焦点环动画](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/focus-ring/)** — Tab 键到达的项目周围会收拢一个圆环。适合用在表单和菜单上。
- **[按钮按压缩放](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/button-press-scale/)** — 按下时缩小，松开时弹回。适合用在主按钮上。
- **[磁吸按钮](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/magnetic-button/)** — 向指针倾斜，然后弹回原位。适合用在一个主按钮上。
- **[开关滑动](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/toggle-switch/)** — 开关打开或关闭时，滑块随之滑动。适合用在设置项上。
- **[点赞爱心迸发](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/heart-burst/)** — 爱心弹出并填满，同时迸出小爱心。适合用在点赞按钮上。
- **[成功彩带](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/success-confetti/)** — 任务完成时，按钮处迸出彩带。适合用在重要时刻。
- **[骨架屏](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/skeleton-loader/)** — 灰色色块在内容加载时占位。适合用在信息流和卡片上。
- **[微光扫过效果](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/shimmer-effect/)** — 一道光带扫过灰色占位块。适合用在加载界面上。
- **[加载旋转指示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/loading-spinner/)** — 六个小图形循环转动，表示正在加载。适合用在短暂等待时。
- **[进度动画](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/progress-animation/)** — 进度条、圆环和步骤依次填满，显示进度。适合用在上传场景。
- **[对勾绘制](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/checkmark-draw/)** — 任务成功后，对勾在圆圈里自己画出来。适合用在表单上。
- **[表单字段变形](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/form-field-morph/)** — 开始输入时，标签上移让开。适合用在注册表单上。
- **[通知徽标脉冲](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/badge-pulse/)** — 图标上的徽标脉冲闪动，吸引注意。适合用在未读消息上。
- **[提示气泡浮现](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/tooltip-reveal/)** — 停顿片刻后，小标签淡入出现。适合用在图标按钮上。
- **[抽屉滑出](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/drawer-slide/)** — 侧边面板在变暗的页面上滑入。适合用在移动端菜单上。
- **[弹窗展开](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/modal-expand/)** — 窗口从你按下的按钮处长出来。适合用在详情查看上。
- **[手风琴展开/收起](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/accordion/)** — 每个问题平滑展开，显示答案。适合用在常见问题页面上。
- **[光标跟随](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/cursor-follower/)** — 一个圆点跟随指针，并反转下方颜色。适合用在作品集上。
- **[错误抖动](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/error-shake/)** — 输入错误时，字段左右抖动。适合用在登录表单上。
- **[滑动删除](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/swipe-to-dismiss/)** — 卡片向侧边拖动后飞出，列表随之收拢。适合用在收件箱上。
- **[汉堡菜单切换](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/hamburger-menu-toggle/)** — 三条横线变成叉号，同时菜单打开。适合用在移动端菜单上。
- **[主题切换变形](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/theme-toggle-morph/)** — 颜色切换到深色时，太阳变成月亮。适合用在主题按钮上。
- **[复制到剪贴板](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/copy-to-clipboard/)** — 「复制」变成对勾和「已复制」，然后再变回来。适合用在代码和链接上。
- **[星级评分](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/star-rating/)** — 星星随指针填满，选中时弹出。适合用在评价上。
- **[轻提示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/toast-notification/)** — 短消息滑入角落，之后自行离开。适合用在更新提醒上。
- **[分段控制器](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/segmented-control/)** — 高亮滑向你选中的选项。适合用在切换视图上。
- **[下拉刷新](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/pull-to-refresh/)** — 向下拉动列表会出现旋转指示，然后出现新内容。适合用在信息流上。
- **[渐变边框动画](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/gradient-border/)** — 一条彩色带绕着卡片或按钮的边缘流动。适合用在精选优惠上。
- **[聚光灯悬停辉光](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/spotlight-hover/)** — 柔和的光晕跟随指针，照亮卡片边缘。适合用在特性网格上。
- **[加入购物车飞入](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/add-to-cart-fly/)** — 商品飞入购物车，购物车抖动并计数加一。适合用在商店里。
- **[拖动排序](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/drag-to-reorder/)** — 拖动一个项目，其他项目滑开让位。适合用在待办清单上。
- **[数字滚动](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/rolling-numbers/)** — 每个数字像里程表一样滚动到新值。适合用在价格上。
- **[正在输入指示](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/typing-indicator/)** — 聊天框里的三个圆点轮流跳动，表示对方正在输入。适合用在聊天应用上。
- **[长按确认](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/hold-to-confirm/)** — 按住按钮时它会逐渐填满，满了才执行操作。适合用在删除按钮上。
- **[搜索框展开](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/expanding-search/)** — 搜索图标展开成输入框，用完后收起。适合用在工具栏上。
- **[按钮加载状态](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/button-loading-states/)** — 按钮变成旋转指示，然后变成对勾或叉号。适合用在保存按钮上。
- **[图片拖尾](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/image-trail/)** — 图片在指针身后接连弹出，然后淡出。适合用在作品集上。
- **[粘性菜单](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/gooey-menu/)** — 小按钮像液体一样从圆形按钮里渗出来。适合用在快捷操作上。
- **[灵动岛](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/dynamic-island/)** — 黑色药丸长成通知，然后再缩回去。适合用在实时更新上。
- **[前后对比滑块](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/before-after-slider/)** — 拖动一条线，横跨两张图片进行对比。适合用在照片编辑上。
- **[放大镜](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/magnifier-lens/)** — 镜头跟随指针，近距离展示图片。适合用在商品照片上。
- **[卡片组滑动](https://matinmonshizadeh.github.io/animation-handbook/animations/04-micro-interactions/card-deck-swipe/)** — 左右滑动顶部卡片，下一张随之升起。适合用在快速选择上。

</details>

<a id="05"></a>

### 05 · 文字与排版 · 19 个动画

<sub>专为文字设计的动画——动态排版、逐字效果、渐变流动和词语变换。</sub>

<details><summary>浏览「文字与排版」的全部 19 个动画</summary>

- **[动态排版](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/kinetic-typography/)** — 词语逐个以各自的方式进出。最适合开场。
- **[打字机效果](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/typewriter-effect/)** — 文字在闪烁光标后逐字打出。最适合短标语。
- **[乱码/故障文字](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/scramble-text/)** — 随机符号从左到右逐个锁定为真实文字。最适合科技标题。
- **[可变字体变形](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/variable-font-morph/)** — 一个词平滑地变粗、变细，还会倾斜。最适合标题。
- **[裁剪揭示文字](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-clip-path-reveal/)** — 标题的每一行依次被揭开。最适合大标题。
- **[跑马灯/滚动条](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/marquee-ticker/)** — 文字横向无缝无限滚动。最适合新闻滚动条。
- **[文字变形](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-morphing/)** — 一个词逐字变成下一个词。最适合短标签。
- **[文字渐变动画](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-gradient-animation/)** — 颜色在静止的文字中流动。最适合标题。
- **[描边填充](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/outline-to-fill/)** — 空心字以擦除或淡入的方式填满颜色。最适合大标题。
- **[入场退场排版](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/enter-exit-typography/)** — 每句短语进来、停留足够读完、再离开。最适合标语。
- **[轮换词轮播](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/rotate-word-carousel/)** — 句子中的一个词不断换成下一个。最适合主视觉标题。
- **[故障文字](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/glitch-text/)** — 文字像信号故障一样撕成红青色条带。最适合醒目标题。
- **[路径文字](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-on-path/)** — 文字沿波浪、弧线或圆形行进。最适合徽章和印章。
- **[波浪文字](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/wavy-text/)** — 波浪逐字滚过整个词。最适合俏皮标题。
- **[荧光笔扫过](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/highlighter-sweep/)** — 记号笔依次扫过关键词的后面。最适合要点。
- **[翻牌器](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/split-flap-board/)** — 像机场翻牌一样，翻牌逐个拼出一个词。最适合主视觉标题。
- **[霓虹闪烁](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/neon-flicker/)** — 发光的字母偶尔像破旧霓虹灯一样闪烁。最适合夜晚主题。
- **[手写描绘](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/handwriting-draw/)** — 一个词像用笔写出来一样一笔一画出现。最适合问候语和签名。
- **[文字粒子](https://matinmonshizadeh.github.io/animation-handbook/animations/05-text-typography/text-particles/)** — 由圆点组成的词被指针打散，又弹回原位。最适合开场。

</details>

<a id="06"></a>

### 06 · 3D 与高级效果 · 28 个动画

<sub>WebGL、着色器、粒子、3D 变换、模拟和其他较重的效果。</sub>

<details><summary>浏览「3D 与高级效果」的全部 28 个动画</summary>

- **[3D 模型环绕](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/3d-model-orbit/)** — 带光照的 3D 形状会自己旋转，或转向你的指针。适合产品展示。
- **[滚动驱动 3D 旋转](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/scroll-driven-3d-rotation/)** — 滚动会让 3D 立方体从一个姿态转到下一个。适合产品导览。
- **[视差 3D 倾斜](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/parallax-3d-tilt/)** — 卡片朝指针倾斜，一道光随之滑过。适合卡片。
- **[Canvas 粒子效果](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/canvas-particle-effect/)** — 圆点漂浮，靠近时连线，还会躲开指针。适合科技类网站。
- **[流体模拟](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/fluid-simulation/)** — 色块像液体一样漂移、融化、合在一起。适合主视觉背景。
- **[动态毛玻璃](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/glassmorphism-animated/)** — 磨砂玻璃卡片模糊了身后的颜色。适合卡片和面板。
- **[WebGL 着色器动画](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/webgl-shader-animation/)** — 逐像素计算的流动色彩图案。适合大胆的背景。
- **[噪声驱动运动](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/noise-based-motion/)** — 平滑噪声让圆点摇摆、色块起伏。适合安静的背景。
- **[SVG 路径动画](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/svg-path-animation/)** — 线条画一笔一笔自己画出来。适合图标和标志。
- **[色差](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/chromatic-aberration/)** — 一个词裂成红、绿、蓝三色边缘。适合醒目的标题。
- **[2.5D / 伪 3D](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/2-5d-pseudo-3d/)** — 平面图层以不同幅度滑动，假装出景深。适合主视觉场景。
- **[光线步进 / SDF 场景](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/ray-marching-sdf/)** — 只用公式画出的 3D 场景，镜头绕着它转。适合艺术页面。
- **[GPGPU 粒子系统](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/gpgpu-particle-system/)** — 数万个粒子在显卡上流动。适合主视觉特效。
- **[悬停图片扭曲](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/image-distortion-hover/)** — 图片在指针周围起涟漪、弯曲。适合作品集图片。
- **[布料模拟](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/cloth-simulation/)** — 一块布在风中飘动，还可以拖着玩。适合有趣的页面。
- **[体积烟雾 / 3D 噪声](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/volumetric-smoke/)** — 柔和的烟雾向上盘旋，像真正的 3D 云。适合有氛围的背景。
- **[变形色块](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/morphing-blob/)** — 色块融在一起，其中一滴追着你的指针。适合主视觉区域。
- **[3D 翻转卡片](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/flip-card-3d/)** — 卡片在 3D 中翻过来，露出背面。适合个人和产品卡片。
- **[3D 轮播](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/carousel-3d/)** — 卡片围成一圈，在 3D 中随拖动旋转。适合图片画廊。
- **[景深照片](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/depth-map-photo/)** — 平面照片随指尖变成 3D：近处移动得更多。适合主视觉图片。
- **[鸟群](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/flocking/)** — 鸟儿成群盘旋、一起转向。适合安静的背景。
- **[ASCII / 半调](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/ascii-halftone/)** — 动态画面用字母或圆点重新画出来。适合复古和科技风格。
- **[2D 物理](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/physics-2d/)** — 球从你点击的地方落下，弹跳、堆积。适合有趣的开场。
- **[液态玻璃](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/liquid-glass/)** — 拖动玻璃面板，身后的画面在边缘弯曲。适合卡片。
- **[全息卡片](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/holographic-card/)** — 卡片随指尖倾斜，彩虹箔片随之闪烁。适合收藏品。
- **[无限缩放](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/infinite-zoom/)** — 视野不断放大，进入画中画。适合页面开场。
- **[水面涟漪](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/water-ripple/)** — 涟漪从指尖扩散，弯曲下面的画面。适合主视觉图片。
- **[音频可视化](https://matinmonshizadeh.github.io/animation-handbook/animations/06-3d-advanced/audio-visualizer/)** — 音条随音乐起舞，从低音到高音。适合音乐播放器。

</details>

<a id="07"></a>

### 07 · 氛围与背景 · 26 个动画

<sub>被动的循环效果，不抢注意力也能保持视觉趣味——渐变、粒子、噪点和辉光。</sub>

<details><summary>浏览「氛围与背景」的全部 26 个动画</summary>

- **[动态渐变背景](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/animated-gradient-background/)** — 颜色在柔和的渐变背景上缓缓流动。最适合主视觉区。
- **[网格渐变](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/mesh-gradient/)** — 模糊的彩色色块漂移、交融。最适合落地页。
- **[极光](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/aurora/)** — 模糊的光带如北极光般摇曳。最适合深色背景。
- **[颗粒噪点叠加](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/grain-overlay/)** — 细腻的颗粒在页面上闪烁，如老电影。最适合编辑类网站。
- **[扫描线效果](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/scanline/)** — 深色线条和扫过的光带，模仿老电视屏幕。最适合复古网站。
- **[漏光](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/light-leak/)** — 温暖的光线不定时地从角落漫入。最适合照片类网站。
- **[星空](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/starfield/)** — 星星从黑暗中向你飞来。最适合太空主题。
- **[呼吸光晕](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/breathing-glow/)** — 柔和的光晕缓缓变大变小，如平静的呼吸。最适合闲置状态。
- **[环境涟漪](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/ambient-ripple/)** — 涟漪从几个点扩散开来，如静水面上的水滴。最适合主视觉区。
- **[漂浮元素](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/floating-elements/)** — 小图形缓缓漂移，各走各的路径。最适合主视觉背景。
- **[网格 / 圆点图案视差](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/grid-dot-pattern-parallax/)** — 圆点网格随指针轻轻移动，营造深度感。最适合科技类网站。
- **[抽象几何动效](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/abstract-geometric-motion/)** — 图形旋转、扩散、流动，形成平静无尽的图案。最适合音乐播放器。
- **[粒子星座](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/particle-constellation/)** — 漂移的圆点靠近时用线连起来。最适合科技类网站。
- **[流场](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/flow-field/)** — 粒子沿看不见的水流前进，留下渐隐的轨迹。最适合艺术页面。
- **[合成波网格](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/synthwave-grid/)** — 发光的网格在条纹太阳下向你滚动而来。最适合音乐和游戏。
- **[矩阵雨](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/matrix-rain/)** — 发光的字符列在深色屏幕上落下。最适合科技主题。
- **[等离子场](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/plasma/)** — 平滑的色彩波浪无尽流动，全部由数学生成。最适合创意网站。
- **[波浪层](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/wave-layers/)** — 层层波浪横向漂移，各自速度不同。最适合主视觉区。
- **[雪 / 雨](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/snow-rain/)** — 雪花飘落，或雨丝落下，洒在黑暗的小镇上。最适合季节性页面。
- **[烟花](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/fireworks/)** — 火箭升空，爆成发光的火花，落下消散。最适合庆祝场合。
- **[等高线](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/topographic-lines/)** — 如徒步地图上的线条，缓缓弯曲漂移。最适合户外品牌。
- **[光束](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/light-rays/)** — 柔和的光束从上方洒落，缓缓扫过。最适合主视觉开场。
- **[生命游戏](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/game-of-life/)** — 细胞按邻居生死，图案不断生长。最适合科技类网站。
- **[肯·伯恩斯效果](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/ken-burns/)** — 照片缓缓缩放平移，再淡入下一张。最适合主视觉幻灯片。
- **[火焰效果](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/fire-effect/)** — 火焰闪烁升腾，随热量升降。最适合游戏和活动页面。
- **[闪电](https://matinmonshizadeh.github.io/animation-handbook/animations/07-ambient-background/lightning/)** — 锯齿状的闪电从云层劈下，照亮天空。最适合戏剧性开场。

</details>

## 设计原则

**一个效果，一个文件夹。**一个 HTML 文件装下演示的标记、CSS 和 JavaScript，再配一篇 README 讲清楚它。演示周围的页面来自两个共享文件，`assets/css/demo-page.css` 和 `assets/js/demo-page.js`。没有构建步骤，没有框架，没有外部依赖。

**先展示，再讲解。**演示本身就是主要的成果。每个效果的 README 是为它服务的。

**讲效果，不讲工具。**GSAP、Framer Motion 和 Three.js 只在生产注意事项里提到——它们永远不是某个条目的主题。

**用大白话，不用代码。**页面用简单的话解释每个动画，并给一句提示词代替代码。代码也还在，想看的人可以在每个页面的源码和 README 里找到。

**人人都能用。**每个页面在手机上、触摸操作和键盘操作下都能用（有效果跟随指针的地方，方向键也能移动它），并且尊重减弱动态效果的设置。

## 本地运行

克隆仓库，用任意静态文件服务器提供它的文件夹，比如 Python 自带的那个：

```bash
git clone https://github.com/matinmonshizadeh/animation-handbook.git
cd animation-handbook
python -m http.server 8000 --bind 127.0.0.1
```

然后在浏览器里打开 http://127.0.0.1:8000/。如果找不到 `python`（很多 macOS 和 Linux 系统上就是这样），用 `python3`。

没有服务器也可以直接从文件夹里打开任意页面的 `index.html`：每个演示都能运行，页面之间的链接也能用。有两个部分需要服务器。每个页面会从自己的 README 里读取*它是什么*和*相似效果*，所以从磁盘打开时，它会在*它是什么*的位置显示一个指向 README 的链接，并略去*相似效果*。另外在首页上，点一张卡片的「复制提示词」按钮，它会变成「打开页面复制」；再点一次会打开这个动画的页面并定位到它的提示词，在那里可以复制。

## 浏览器支持

手册面向 Chrome、Edge、Firefox 和 Safari 的当前版本。少数页面依赖的特性在旧浏览器或某些设备上可能没有。没有这些特性时，每个页面的表现如下：

- **WebGL。**3D 与高级效果里有八个页面用它绘制：3D 模型环绕、景深照片、流体模拟、GPGPU 粒子系统、悬停图片扭曲、光线步进 / SDF 场景、体积烟雾 / 3D 噪声和 WebGL 着色器动画。没有它时，景深照片改在普通的 2D 画布上移动图层，另外七个页面在动画的位置显示一则简短的说明。GPGPU 粒子系统需要支持浮点颜色缓冲区的 WebGL 2，没有时会显示它的说明。
- **View Transitions API。**浏览器没有它时，View Transitions API 页面无论选了哪种过渡样式和速度，页面之间都用简单的淡入淡出切换。
- **CSS 滚动驱动动画。**只有层叠卡片用它，在下一张卡片盖上来时把每张卡片缩小。没有它时，一小段脚本会在滚动时做同样的事。
- **磨砂玻璃（CSS `backdrop-filter`）。**动态毛玻璃和液态玻璃就是靠它做的，滚动侦测导航、七个氛围与背景演示，以及每个动画页面的顶栏，都用它在菜单、文字卡片或顶栏后面做轻微模糊。没有它时，每个面板仍保留半透明的底色，但后面的东西不会模糊；动态毛玻璃的「磨砂」设置的「关」选项展示的就是这个样子。
- **液态玻璃的弯曲。**液态玻璃还会在边缘弯曲后面的图片，但只在电脑或 Android 上的 Chrome、Edge 和其他 Chromium 浏览器里有效；在别处它就是磨砂玻璃。

## 参与贡献

欢迎贡献——新效果、修复和改进。每个动画都在 `animations/<分类>/<名字>/` 下，有自己的 `index.html` 和 `README.md`，还可以有一张预览图。创作指南见 [CONTRIBUTING.md](CONTRIBUTING.md)，完整规范见 [CLAUDE.md](CLAUDE.md)，后者是写给 AI 编程助手看的。

有问题、想分享点子，请去 [Discussions](https://github.com/matinmonshizadeh/animation-handbook/discussions)。想看更新内容，见 [CHANGELOG.md](CHANGELOG.md)。

如果这本手册帮到了你，一个 ⭐ 能让更多人找到它。

## 致谢

网站字体是 Schibsted Grotesk，作者为 The Schibsted-Grotesk Project Authors，遵循 SIL Open Font License 1.1——见 [assets/fonts/OFL.txt](assets/fonts/OFL.txt)。

## 许可

[MIT](LICENSE) © 2026 Matin Monshizadeh。字体遵循 SIL Open Font License（见[致谢](#致谢））。
