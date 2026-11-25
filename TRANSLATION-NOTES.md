# 动画手册 · 汉化说明（TRANSLATION-NOTES.md）

本文档记录 animation-handbook（原 `matinmonshizadeh/animation-handbook`，MIT）汉化为中文（zh-CN）的规则与术语表。汉化工作在本地分支 `zh-CN` 上进行，不 fork、不 push、不联系 GitHub。

## 总规则

1. **翻译所有可见文字**：界面文案、说明、README、现成的 AI 提示词（提示词翻成中文提示词，用户可直接复制给中文 AI 助手使用）。
2. **保持英文不变**：代码标识符、CSS 属性/属性值、class 名、URL、文件名、动画 slug（`parallax-scrolling` 等）、HTML 属性值中的代码、`lang` 之外的语言代码、shields.io 徽章 URL、GitHub 仓库地址。
3. **一处翻译，处处一致**：每个动画的一行简介（"Best for …" 句）出现在多处——演示页 h1 下的 lede、meta/OG/JSON-LD、根目录 `index.html` 的 CATS 数组、动画 README、分类 README、根 README 的 Full catalog——**必须在所有地方用完全相同的中文译文**。术语表（见下）是唯一事实来源。
4. **不编造内容，不改演示行为**：只翻译，不重写、不增删功能；提示词里的 `[方括号占位]` 保留方括号，翻译括号内的词。
5. **按钮名互相引用时一致**：正文、帮助行里提到按钮时用术语表里的中文按钮名，并加「」标明，例如「按「播放」」。
6. **跳过 `docs/superpowers/`**（开发设计文档，不面向用户）。
7. **测试与内容同步改**：测试里逐字断言 UI 文案（见 `tests/*.test.js`），改文案时同步改测试的期望字符串；改完跑 `node --test "tests/*.test.js"` 和 `node tools/check-pages.mjs`。
8. **语言属性**：所有 HTML 页 `lang="en"` → `lang="zh-CN"`；JSON-LD `"inLanguage": "en"` → `"zh-CN"`。
9. **中文标点**：中文句子用中文标点（，。：；「」等）；但代码、URL、CSS 值、JSON 里的英文标点不动。`data-hb-motion-note` 的值是中文句子，**不以句号结尾**（测试用正则 `/^[^。]*[^。\s]$/` 之类的中文规则校验，见 tests）。

## 站名

| 英文 | 中文 | 说明 |
|---|---|---|
| Animation Handbook | 动画手册 | 站名；仓库名、URL、路径里的 `animation-handbook` 不动 |
| Famous effects | 知名网站效果 | famous/ 页标题；导航短名用「知名效果」 |

## 分类名（CATS / README / 分类页标题共用）

| 文件夹 | 英文 | 中文 |
|---|---|---|
| 01-scroll-based | Scroll-Based | 滚动驱动 |
| 02-entrance-and-exit | Entrance & Exit | 入场与退场 |
| 03-page-transitions | Page Transitions | 页面切换 |
| 04-micro-interactions | Micro-Interactions | 微交互 |
| 05-text-typography | Text & Typography | 文字与排版 |
| 06-3d-advanced | 3D & Advanced | 3D 与高级效果 |
| 07-ambient-background | Ambient & Background | 氛围与背景 |

分类简介（cat.desc，CATS / 根 README / 分类页共用）：

- 01 滚动驱动：由滚动位置驱动的动画——视差、吸顶、擦除、吸附和滚动叙事。
- 02 入场与退场：元素级的入场与退场动画——淡入、滑动、揭示、翻转和文字依次出现。
- 03 页面切换：路由或视图之间的整页切换——交叉淡入淡出、滑动、传送门、变形和 View Transitions API。
- 04 微交互：短暂的、由用户触发的动画——悬停、点击、聚焦、加载状态和界面反馈模式。
- 05 文字与排版：专为文字设计的动画——动态排版、逐字效果、渐变流动和词语变换。
- 06 3D 与高级效果：WebGL、着色器、粒子、3D 变换、模拟和其他较重的效果。
- 07 氛围与背景：被动的循环效果，不抢注意力也能保持视觉趣味——渐变、粒子、噪点和辉光。

## 共享界面术语（176 个演示页 + demo-page.js + 首页共用）

| 英文 | 中文 | 出现位置 |
|---|---|---|
| Pause | 暂停 | 暂停按钮 `.hb-pause-label` |
| Play | 播放 | 播放按钮 |
| Replay | 重播 | 重播按钮（once 类页面） |
| Show me | 演示一下 | 演示按钮 |
| Reset | 重置 | 重置按钮 |
| Back to top | 回到顶部 | 回到顶部按钮 / 页脚链接（页脚带 ↑） |
| Loop | 循环 | 循环复选框 |
| Slow motion | 慢动作 | 慢动作复选框 |
| Normal | 普通 | 模式按钮 |
| See inside | 看内部 | 模式按钮 |
| Compare | 对比 | 模式按钮 |
| What the box shows（aria-label） | 画面显示内容 | 模式组 aria-label |
| Star on GitHub | 在 GitHub 上点星 | 星标行按钮（aria-label 与可见文字） |
| Source on GitHub | GitHub 上的源码 | 页脚链接 |
| Free and open source. If this helped, a star on GitHub helps other people find it. | 免费开源。如果你觉得有用，在 GitHub 上点个星星，能帮更多人找到它。 | 星标行段落 |
| Copy prompt | 复制提示词 | 复制按钮（演示页 `.hb-copy-label` 与首页卡片 `.copy-l`） |
| Copied | 已复制 | 复制成功后的按钮文字 |
| Open to copy | 打开页面复制 | 首页卡片复制失败时的按钮文字（剪贴板不可用时） |
| Show the full prompt | 显示完整提示词 | 提示词过长时的展开按钮（demo-page.js） |
| Show less | 收起 | 提示词展开后的收起按钮 |
| Press ⌘C to copy | 按 ⌘C 复制 | 提示词展开后的复制提示（macOS） |
| Press Ctrl+C to copy | 按 Ctrl+C 复制 | 提示词展开后的复制提示（其他系统） |
| Settings from the demo: | 演示中的设置： | 复制提示词时附加的设置行前缀（demo-page.js `promptWithSettings`） |
| More options | 更多选项 | 设置展开按钮 |
| Fewer options | 收起选项 | 设置收起按钮 |
| Your settings | 你的设置 | 设置 chips 标题 |
| Replace the words in brackets with your own element. Your settings are added at the end. | 把方括号里的词换成你自己的元素。你的设置会加在末尾。 | 提示词下方 hint |
| Match the settings listed below. | 按下面列出的设置来做。 | 提示词结尾句（170 个带 Try it 的页面） |
| on / off（复选框值） | 开 / 关 | 设置 chips 里的值（demo-page.js `valueFor`） |

### 步骤标题（按页面类型）

| 英文 | 中文 |
|---|---|
| Watch it | 看效果 |
| Hover it | 悬停试试 |
| Click it | 点击试试 |
| Drag it | 拖动试试 |
| Scroll it | 滚动试试 |
| Press Tab | 按 Tab 试试 |
| Try it | 试一试 |
| Copy the prompt | 复制提示词 |

### About 区标题

| 英文 | 中文 |
|---|---|
| What it is | 它是什么 |
| Good for | 适合用在 |
| Avoid on | 避免用在 |
| Similar animations | 相似效果 |

### 上一页 / 下一页

- `Previous: ` → `上一个：`（可见文字与 aria-label 都改；aria-label 里的动画名用中文名）
- `Next: ` → `下一个：`

### 减弱动态提示（demo-page.js `motionNote`，中文拼接规则）

- 循环 / 慢动作关闭部分：`循环`、`慢动作`，两项用「和」连接，后接 `已关闭`，例如 `循环和慢动作已关闭`、`循环已关闭`。
- 滚动类页面默认句：`效果随滚动变化，不播放动画`
- 暂停类页面：`开始时处于暂停状态`
- 各部分用 `，` 连接，末尾加 `，因为你的设备设置了减弱动态效果。`
- 例：`循环和慢动作已关闭，因为你的设备设置了减弱动态效果。`；`开始时处于暂停状态，慢动作已关闭，因为你的设备设置了减弱动态效果。`

### 复制提示词的 aria-label（首页卡片）

- `Copy prompt for <名>` → `复制「<中文名>」的提示词`
- `Prompt copied for <名>` → `已复制「<中文名>」的提示词`
- `Open to copy the prompt for <名>` → `打开页面复制「<中文名>」的提示词`

## README 章节标题（动画 README 与 demo-page.js 的 `sections()` 共用）

| 英文 | 中文 |
|---|---|
| What it is | 它是什么 |
| When to use it | 什么时候用 |
| How it works | 工作原理 |
| Key parameters | 关键参数 |
| Production notes | 生产注意事项 |
| See also | 另见 |
| Live demo | 在线演示 |

Key parameters 表头：`| Parameter | Default | Effect |` → `| 参数 | 默认值 | 效果 |`

README 标题 `# <英文名>` → `# <中文名>`；`[Live demo](…)` → `[在线演示](…)`；正文里的代码、属性名、CSS 值、URL 不动；"See also" 里的动画名用中文名，链接不变。

## 首页（根 index.html）术语

- 搜索框占位 `Search animations` → `搜索动画效果`；搜索按钮 `Search` → `搜索`
- 搜索框标签 `Describe the animation you want` → `描述你想要的动画效果`
- `Good places to start` → `从这里开始`
- `Eight favorites to try first. Type above, or pick a group.` → `先试试这八个精选。可以在上面输入，或选一个分组。`
- `Find an animation you saw` → `找到你见过的动画`
- `What moved?` → `什么动了？`；`When did it move?` → `什么时候动的？`；`How did it move?` → `怎么动的？`
- `Pick the picture closest to what you saw.` → `选一张最接近你看到的图。`
- `Not sure` → `不确定`；`See N matches` → `查看 N 个匹配`；`Start over` → `重新开始`；`Not here?` → `没找到？`
- `Best matches` → `最匹配`；`Close matches` → `接近的匹配`
- `They fit all but one of your answers.` → `只差一个答案就完全符合。`
- `Show all` → `显示全部`；`Show all N matches` → `显示全部 N 个匹配`；`Show all N` → `显示全部 N 个`
- `How it works` → `怎么用`；`Three steps, and no code to read.` → `三步，不用读代码。`
- `All animations` → `全部动画`；`N animations in M groups.` → `共 N 个动画，分 M 组。`；分组计数 `N animations` → `N 个动画`
- `Famous effects`（首页分组）→ `知名效果`
- 首页卡片上的 `Copy prompt` / `Copied` / `Open to copy` 见共享术语表。
- 搜索分词：`wordsOf` 要保留中文字符（单字 + 连续中文串），英文行为不变；`stem` 只处理英文，对中文 token 无害。
- FIND_Q 的 `words`（显示词）翻成中文，`key` 保持英文小写（404 页与测试依赖 key 的格式）。

### FIND_Q 答案词

| 英文 | 中文 |
|---|---|
| Text or numbers | 文字或数字 |
| A picture or card | 图片或卡片 |
| A button or icon | 按钮或图标 |
| A menu or form | 菜单或表单 |
| A loading sign or message | 加载标志或提示 |
| A line or shape | 线条或图形 |
| Dots or particles | 圆点或粒子 |
| The background | 背景 |
| The whole page | 整个页面 |
| When I scrolled | 滚动时 |
| When I pointed at it | 指向它时 |
| When I clicked or tapped | 点击或轻触时 |
| When I dragged or swiped | 拖动或滑动时 |
| When the page opened | 页面打开时 |
| When I went to another page | 跳到另一页时 |
| By itself, again and again | 自己不停地动 |
| Faded or blurred | 淡入淡出或变模糊 |
| Slid | 滑动 |
| Grew or shrank | 变大或变小 |
| Turned or flipped | 旋转或翻转 |
| Bounced or shook | 弹跳或抖动 |
| One after another | 一个接一个 |
| Broke into pieces | 碎成一片片 |
| Followed my pointer | 跟着指针走 |
| Changed color or glowed | 变色或发光 |
| Changed into something else | 变成了别的东西 |
| Drew or typed itself | 自己画出来或打出来 |
| Got uncovered | 被揭开 |
| Moved in layers | 分层移动 |
| Stayed in place while I scrolled | 滚动时停在原地 |
| Drifted or floated | 漂移或悬浮 |
| Rippled or waved | 起涟漪或波动 |
| Glitched or flickered | 故障闪烁 |

### PLACES（首页分组入口）

| 英文 | 中文 |
|---|---|
| Buttons | 按钮 |
| Text | 文字 |
| Images & cards | 图片和卡片 |
| Backgrounds | 背景 |
| Menus & forms | 菜单和表单 |
| Loading & messages | 加载和提示 |
| Page intros | 页面开场 |
| Scrolling | 滚动 |
| Page changes | 页面切换 |

## 404 页

- `Error 404` → `错误 404`
- `Go to the home page` → `回到首页`
- `Browse all 176 animations` → `浏览全部 176 个动画`（数字随总数变）
- `Find an animation by what you saw` → `按你见过的样子找动画`
- 页面其他文案（标题、说明、搜索框）按同一术语表翻译。

## 根 README 格式（测试会逐字校验格式，见 tests/pages.test.js）

- 标题与简介里的数字保持与 CATS 一致；`176 web animation techniques` → `176 个网页动画效果`。
- Full catalog 标题格式：`### 01 · 滚动驱动 · 27 个动画`（`### NN · 中文分类名 · N 个动画`）。
- 分类下简介行：`<sub>分类中文简介</sub>`，与 CATS 里的 cat.desc 完全一致。
- 展开摘要：`<details><summary>浏览「滚动驱动」的全部 27 个动画</summary>`（格式：`浏览「中文分类名」的全部 N 个动画`）。
- 条目格式不变：`- **[中文名](https://animation.xizanghuapi.cn/animations/…/)** — 中文简介`，简介与 CATS 的 desc 完全一致。
- `All 176 techniques, each linked to its live demo.` → `全部 176 个动画，每个都有在线演示链接。`
- Categories 表：分类名用中文，链接不变，计数不变。

## 工具脚本里的硬编码英文

- `tools/update-sitemap.mjs` 的 llms.txt 开头文案是硬编码英文（`# Animation Handbook`、`> A visual reference of…`、`Each page plays one technique live…`、`## Famous effects`、`## Optional`、`- [Source on GitHub]…`），改工具本身的文案再重新生成，否则重新生成会把 llms.txt 翻回英文。
- `tools/make-famous.mjs` 从首页重新生成 famous 页的生成块（PREVIEWS 与预览 CSS），**在 CATS 翻译完成后再运行**。
- `tools/update-sitemap.mjs` 重新生成 llms.txt、sitemap.xml 与首页 no-JS  plain-links，**在 CATS 翻译完成后再运行**。

## 动画术语表（176 个动画的中文名 + 一行中文简介）

由各分类负责人起草、协调人汇总后填入本节。规则：中文名要自然、符合中文前端开发者的习惯说法；简介是 "Best for …" 句的完整翻译；slug 保持英文；简介在 CATS、演示页、README、分类 README、根 README 五处完全一致。

### 01 滚动驱动（27）—— 已翻译（2026-11-23；分类负责人结束时未交报告，术语表由协调人从文件提取并用脚本校验）

| slug | 中文名 | 中文简介 |
|---|---|---|
| counter-animation | 数字滚动计数 | 数字滚动进入视野时开始累加。最适合统计数据。 |
| cover-card-to-fixed-header | 封面卡片变固定顶栏 | 高封面滚动时收成细顶栏。最适合文章页。 |
| fly-in-fly-out-contact-list | 联系人列表飞入飞出 | 行在靠近顶部或底部边缘时淡入淡出、滑入滑出。最适合长列表。 |
| hide-on-scroll-header | 滚动隐藏顶栏 | 往下滚隐藏顶栏，往上滚显示出来。最适合移动端页面。 |
| horizontal-scroll | 横向滚动 | 往下滚，一行面板横向滑过。最适合作品集。 |
| parallax-depth-of-field | 视差景深 | 各图层随焦点变化以各自的速度移动。最适合电影感开场。 |
| parallax-scrolling | 视差滚动 | 远处的图层比近处的移动得更慢。最适合主视觉场景。 |
| pin-animation | 固定动画 | 一部分固定不动，文字随滚动变化。最适合功能列表。 |
| progress-bar | 阅读进度条 | 往下读，进度条随之填满。最适合长文章。 |
| reveal-on-scroll | 滚动显现 | 卡片越过盒子里的一条线时出现。最适合长落地页。 |
| reverse-scrolling-columns | 反向滚动列 | 两侧的列与中间一列朝相反方向滚动。最适合作品集。 |
| scroll-background-color | 滚动驱动背景色 | 背景色随滚动经过的区域而变化。最适合故事讲述。 |
| scroll-image-sequence | 滚动图片序列 | 滚动像翻书一样播放一串图片。最适合产品展示。 |
| scroll-trigger | 滚动触发动画 | 动画在设定的滚动点开始、跟随并固定。最适合落地页。 |
| scroll-velocity-skew | 滚动速度倾斜 | 滚得快时行会倾斜，停下时回正。最适合画廊。 |
| scrollspy-nav | 滚动侦测导航 | 菜单高亮你正在读的章节。最适合长文档。 |
| scrollytelling | 滚动叙事 | 旁边的图片随你往下读而变化。最适合数据故事。 |
| scrub-animation | 滚动 scrub 动画 | 滚动像拖视频进度条一样正放倒放。最适合产品演示。 |
| section-wipe | 区域擦除切换 | 每个区域从上一个上面滑过。最适合全屏故事。 |
| smooth-scroll | 平滑（惯性）滚动 | 滚动滑到停下，而不是一下跳到位。最适合作品集网站。 |
| snap-scrolling | 滚动吸附 | 盒子一次停在一个完整区域上。最适合幻灯片和画廊。 |
| stacking-cards | 层叠卡片 | 卡片吸顶并叠成一摞。最适合分步讲述。 |
| stagger-reveal | 交错显现 | 一组里的元素随滚动依次出现。最适合卡片网格。 |
| sticky-section | 吸顶区域 | 一个区域固定不动，内容随之变化。最适合功能导览。 |
| svg-line-draw | SVG 线条滚动描绘 | 线条随滚动沿路线自己画出来。最适合时间线。 |
| text-fill-on-scroll | 滚动文字填充 | 词语随滚动一个接一个亮起来。最适合关键语句。 |
| zoom-into-image | 图片缩放进入 | 小窗口随滚动放大填满整个盒子。最适合主视觉图片。 |

01 备注：协调人校验通过——27 页 JSON-LD 均解析，name = h1，articleSection = 滚动驱动，inLanguage = zh-CN；meta/og 描述与 lede 一致；27 个 README 标题 = h1，含 [在线演示]，六个中文章节标题，`| 参数 | 默认值 | 效果 |` 表；上一页/下一页可见文案含邻居新 h1；8 条 `data-hb-motion-note` 均为中文且无句号（符合待改的测试正则 `/^[^。.]*[^。.\s]$/`，翻转时改 `tests/pages.test.js:143`）；6 页提示词本来就没有“按下面列出的设置来做。”结尾（horizontal-scroll、parallax-depth-of-field、pin-animation、scroll-trigger、scrollytelling、sticky-section——HEAD 里也没有，是无设置页，不是翻译丢失）。负责人报告的跨分类 "See also" 保留英文名 9 个，待协调人统一改：counter-animation → Rolling Numbers（04）；parallax-scrolling → Wave Layers（07）；scroll-background-color → Animated Gradient Background（07）；scroll-velocity-skew → Marquee / Ticker（05）；snap-scrolling → 3D Carousel（06）；svg-line-draw → SVG Path Animation（06）；text-fill-on-scroll → Kinetic Typography（05）、Highlighter Sweep（05）；zoom-into-image → Infinite Zoom（06）。行为适配（负责人报告）：scroll-image-sequence 的 `capCount.textContent !== words` 等字面量比较两侧同步翻译；scrollytelling 的 aria-label 由 `' meter deep' / ' meters deep'` 三元改为 `depNum.textContent + ' 米深'`；scroll-trigger 的减弱动态说明由 `FADES.replace('淡入','直接出现')` 处理；progress-bar 的比较说明须含 `NAMES.top` 原文（`NOTE.replace(NAMES.top, NAMES[s])`）；counter-animation 的秒数后缀改为 `'约' + … + '秒'`；text-fill-on-scroll 的演示段落用空格分词（48 个 token）；zoom-into-image 的 HTML 默认说明改为与脚本一致（`窗口：盒子的五分之二`）。已修：3 页 JSON-LD `headline` 替换串多余 `">`（reveal-on-scroll、reverse-scrolling-columns、scroll-background-color）、counter-animation README 的 U+FFFD 坏字符。分类 README 完整（27 条，CATS 顺序），只剩 `## Entries` 标题仍英文（汇总时统一为 `## 全部动画`；负责人建议 `## 条目`，协调人定为 `## 全部动画`，与首页“全部动画”用语一致）。

### 02 入场与退场（15）—— 已翻译（2026-11-23，分类负责人已报完；待协调人汇总进 CATS）

| slug | 中文名 | 中文简介 |
|---|---|---|
| fade-in-out | 淡入淡出 | 淡入出现，淡出离开。适合弹窗和提示框。 |
| slide-in | 滑入 | 从一边滑入到位。适合侧边面板和通知。 |
| slide-up-reveal | 上滑揭示 | 文字从一条看不见的边缘后方升起进入视野。适合标题。 |
| scale-in | 缩放进入 | 从小放大到完整尺寸。适合弹窗和菜单。 |
| clip-path-reveal | 剪裁路径揭示 | 一个不断变大的形状揭开它，自身保持静止。适合图片和横幅。 |
| curtain-reveal | 幕布揭示 | 一块彩色面板先盖住它，随后滑走。适合开场和 Logo。 |
| split-text-reveal | 拆字揭示 | 文字拆成碎片，依次出现。适合标题。 |
| letter-by-letter-stagger | 逐字母出现 | 一句话逐字母出现，或像打字机一样。适合短标题。 |
| word-by-word-reveal | 逐词揭示 | 词语按阅读节奏依次出现。适合引言和开场。 |
| blur-in | 模糊渐入 | 像相机对焦一样从模糊中变得清晰。适合精选卡片和照片。 |
| flip-in | 翻转进入 | 像卡片翻面一样以 3D 方式翻入视野。适合卡片和瓷贴。 |
| bounce-in | 弹跳进入 | 带着弹簧般的弹跳落定。适合徽章和成功提示。 |
| rotate-in | 旋转进入 | 边旋转边放大进入到位。适合图标、星星和徽章。 |
| shatter-effect | 碎裂效果 | 碎成碎片飞散并淡出。适合画廊和醒目的退场。 |
| ink-spread-reveal | 墨水扩散揭示 | 墨滴扩散、汇合，直至画面显现。适合艺术和开场。 |

02 备注：演示内容本身是演示对象时保留英文——letter-by-letter 的输入 `Hello, world.`、split-text 的输入 `看见显而易见之外`、slide-up-reveal 的标题 "Beautiful / interfaces / 用心做好。"、word-by-word 的例句（按空格分词，中文会破坏演示）、shatter 的照片标题、clip-path 的表面标签与标题 "Unmasked"、curtain 的徽章/标题 "Unveiled" 及其副标题。行为适配：word-by-word 的时长助手按中文语序重写（`NUM` → 一/两/三…，`n<1` → 半秒）；curtain-reveal 的无暂停正则 `.replace(/,? (?:and)?holds?,/,',')` 改为中文 `.replace(/，?停顿一下/,'')`，四条模式说明都写成含“，停顿一下”以便移除后仍通顺；flip-in 的 JS 变量 `edge` 保持英文（top/bottom/left），模板字符串映射为“卡片的上/下/左 边”；各页时长词助手按中文语序重组（"about a third of a second" → 约三分之一秒；"a second and a half" → 一秒半）。跨分类 "See also" 保留英文名，待协调人统一改：curtain-reveal → Overlay Wipe（03）；shatter-effect → Dissolve Transition（03）、2D Physics（06）；ink-spread-reveal → Circle Reveal（03）。分类 README 的 `## Entries` 标题译为 `## 全部动画`（协调人统一：01/03 仍是 `## Entries`，汇总时统一改为 `## 全部动画`）。

### 04 微交互（44）—— 已翻译（2026-11-23；负责人已报告，协调人校验 0 问题，术语表与报告完全一致）

| slug | 中文名 | 中文简介 |
|---|---|---|
| accordion | 手风琴展开/收起 | 每个问题平滑展开，显示答案。适合用在常见问题页面上。 |
| add-to-cart-fly | 加入购物车飞入 | 商品飞入购物车，购物车抖动并计数加一。适合用在商店里。 |
| badge-pulse | 通知徽标脉冲 | 图标上的徽标脉冲闪动，吸引注意。适合用在未读消息上。 |
| before-after-slider | 前后对比滑块 | 拖动一条线，横跨两张图片进行对比。适合用在照片编辑上。 |
| button-loading-states | 按钮加载状态 | 按钮变成旋转指示，然后变成对勾或叉号。适合用在保存按钮上。 |
| button-press-scale | 按钮按压缩放 | 按下时缩小，松开时弹回。适合用在主按钮上。 |
| card-deck-swipe | 卡片组滑动 | 左右滑动顶部卡片，下一张随之升起。适合用在快速选择上。 |
| checkmark-draw | 对勾绘制 | 任务成功后，对勾在圆圈里自己画出来。适合用在表单上。 |
| click-ripple | 点击涟漪 | 按下的位置会扩散出一圈涟漪。适合用在按钮和列表项上。 |
| copy-to-clipboard | 复制到剪贴板 | 「复制」变成对勾和「已复制」，然后再变回来。适合用在代码和链接上。 |
| cursor-follower | 光标跟随 | 一个圆点跟随指针，并反转下方颜色。适合用在作品集上。 |
| drag-to-reorder | 拖动排序 | 拖动一个项目，其他项目滑开让位。适合用在待办清单上。 |
| drawer-slide | 抽屉滑出 | 侧边面板在变暗的页面上滑入。适合用在移动端菜单上。 |
| dynamic-island | 灵动岛 | 黑色药丸长成通知，然后再缩回去。适合用在实时更新上。 |
| error-shake | 错误抖动 | 输入错误时，字段左右抖动。适合用在登录表单上。 |
| expanding-search | 搜索框展开 | 搜索图标展开成输入框，用完后收起。适合用在工具栏上。 |
| focus-ring | 焦点环动画 | Tab 键到达的项目周围会收拢一个圆环。适合用在表单和菜单上。 |
| form-field-morph | 表单字段变形 | 开始输入时，标签上移让开。适合用在注册表单上。 |
| gooey-menu | 粘性菜单 | 小按钮像液体一样从圆形按钮里渗出来。适合用在快捷操作上。 |
| gradient-border | 渐变边框动画 | 一条彩色带绕着卡片或按钮的边缘流动。适合用在精选优惠上。 |
| hamburger-menu-toggle | 汉堡菜单切换 | 三条横线变成叉号，同时菜单打开。适合用在移动端菜单上。 |
| heart-burst | 点赞爱心迸发 | 爱心弹出并填满，同时迸出小爱心。适合用在点赞按钮上。 |
| hold-to-confirm | 长按确认 | 按住按钮时它会逐渐填满，满了才执行操作。适合用在删除按钮上。 |
| hover-state | 悬停状态动画 | 指向卡片时，它有六种反应方式。适合用在按钮和卡片上。 |
| image-trail | 图片拖尾 | 图片在指针身后接连弹出，然后淡出。适合用在作品集上。 |
| loading-spinner | 加载旋转指示 | 六个小图形循环转动，表示正在加载。适合用在短暂等待时。 |
| magnetic-button | 磁吸按钮 | 向指针倾斜，然后弹回原位。适合用在一个主按钮上。 |
| magnifier-lens | 放大镜 | 镜头跟随指针，近距离展示图片。适合用在商品照片上。 |
| modal-expand | 弹窗展开 | 窗口从你按下的按钮处长出来。适合用在详情查看上。 |
| progress-animation | 进度动画 | 进度条、圆环和步骤依次填满，显示进度。适合用在上传场景。 |
| pull-to-refresh | 下拉刷新 | 向下拉动列表会出现旋转指示，然后出现新内容。适合用在信息流上。 |
| rolling-numbers | 数字滚动 | 每个数字像里程表一样滚动到新值。适合用在价格上。 |
| segmented-control | 分段控制器 | 高亮滑向你选中的选项。适合用在切换视图上。 |
| shimmer-effect | 微光扫过效果 | 一道光带扫过灰色占位块。适合用在加载界面上。 |
| skeleton-loader | 骨架屏 | 灰色色块在内容加载时占位。适合用在信息流和卡片上。 |
| spotlight-hover | 聚光灯悬停辉光 | 柔和的光晕跟随指针，照亮卡片边缘。适合用在特性网格上。 |
| star-rating | 星级评分 | 星星随指针填满，选中时弹出。适合用在评价上。 |
| success-confetti | 成功彩带 | 任务完成时，按钮处迸出彩带。适合用在重要时刻。 |
| swipe-to-dismiss | 滑动删除 | 卡片向侧边拖动后飞出，列表随之收拢。适合用在收件箱上。 |
| theme-toggle-morph | 主题切换变形 | 颜色切换到深色时，太阳变成月亮。适合用在主题按钮上。 |
| toast-notification | 轻提示 | 短消息滑入角落，之后自行离开。适合用在更新提醒上。 |
| toggle-switch | 开关滑动 | 开关打开或关闭时，滑块随之滑动。适合用在设置项上。 |
| tooltip-reveal | 提示气泡浮现 | 停顿片刻后，小标签淡入出现。适合用在图标按钮上。 |
| typing-indicator | 正在输入指示 | 聊天框里的三个圆点轮流跳动，表示对方正在输入。适合用在聊天应用上。 |

04 备注：负责人报告 + 协调人校验 0 问题（JSON-LD、meta/og 描述、提示词结尾、README 标题/章节/参数表、上一页/下一页）。负责人修过：44 个 h1 曾漏翻（批量脚本只翻了 title/meta/lede/JSON-LD），补翻后 0 失败；4 个选项按钮与 1 个设置名漏翻（drawer-slide 深、heart-burst 远、shimmer-effect 明亮、success-confetti 很多、Color 设置→颜色）；README 与页面选项词不一致 9 处（已统一）。**协调人修了 2 处翻译引入的脚本语法错误**：progress-animation 的 `,all='the bar, the ring and the steps'` 被改成 `const all='横条、圆环和步骤'`（丢了逗号、多出 const）→ 恢复为 `,all='横条、圆环和步骤'`（`all` 片段须保持为 inside 说明与 `STILL.inside` 的连续子串，修后仍满足）；card-deck-swipe 的 else 分支字符串拼接断裂（`够远到能+(verb===…`）→ 改为 `够远到能'+(verb==='fade'?'淡出。':'飞走。')`。修后 176 页内联脚本全部通过语法检查。04 的 README "另见"跨分类英文名（负责人报告与协调人 grep 一致，汇总时用 h1 映射脚本统一改）：FLIP Technique（accordion、drag-to-reorder、modal-expand）、Shared Element Transition（add-to-cart-fly）、Morph Transition（dynamic-island）、Liquid Glass（before-after-slider、magnifier-lens）、Clip-Path Reveal（before-after-slider）、3D Carousel（card-deck-swipe）、Morphing Blob（gooey-menu）、Text Gradient Animation（gradient-border）、Animated Gradient Background（gradient-border）、Image Distortion on Hover（image-trail）、Progress Bar（progress-animation）、Counter Animation / Rotate Word Carousel / Split-Flap Board（rolling-numbers）、Parallax 3D Tilt（spotlight-hover）、Fireworks（success-confetti）、Wavy Text（typing-indicator）。行为约束：copy-to-clipboard 的演示按钮 `Copy`/`Copied!` 可安全翻译（脚本不比较 textContent）；modal-expand 的 `data-label` 值（Top Left 等）保留英文（JS 标识符比较）；卡片组人名音译（Mira→米拉等），LIKE/NOPE→喜欢/不喜欢；演示密码 `letmein`、虚构品牌（lumen、NORDVEL 表盘、Atlas Studio 等）、按键名 Tab/Enter/Esc、产品名 ChatGPT/Claude/Cursor 保留英文。分类 README 待汇总规范化：标题 `# 04 — 微交互` → `# 微交互`；条目为表格（`## 动画` + `| 演示 | 简介 |`，负责人称表格是原文结构故保留）→ 转为列表并改标题为 `## 全部动画`；`## 关键概念` 已翻译；`## 另见` 的分类名仍英文（02 — Entrance & Exit、03 — Page Transitions）→ 改中文名。

### 05 文字与排版（19）—— 已翻译（2026-11-23，分类负责人已报完；待协调人汇总进 CATS）

| slug | 中文名 | 中文简介 |
|---|---|---|
| kinetic-typography | 动态排版 | 词语逐个以各自的方式进出。最适合开场。 |
| typewriter-effect | 打字机效果 | 文字在闪烁光标后逐字打出。最适合短标语。 |
| scramble-text | 乱码/故障文字 | 随机符号从左到右逐个锁定为真实文字。最适合科技标题。 |
| variable-font-morph | 可变字体变形 | 一个词平滑地变粗、变细，还会倾斜。最适合标题。 |
| text-clip-path-reveal | 裁剪揭示文字 | 标题的每一行依次被揭开。最适合大标题。 |
| marquee-ticker | 跑马灯/滚动条 | 文字横向无缝无限滚动。最适合新闻滚动条。 |
| text-morphing | 文字变形 | 一个词逐字变成下一个词。最适合短标签。 |
| text-gradient-animation | 文字渐变动画 | 颜色在静止的文字中流动。最适合标题。 |
| outline-to-fill | 描边填充 | 空心字以擦除或淡入的方式填满颜色。最适合大标题。 |
| enter-exit-typography | 入场退场排版 | 每句短语进来、停留足够读完、再离开。最适合标语。 |
| rotate-word-carousel | 轮换词轮播 | 句子中的一个词不断换成下一个。最适合主视觉标题。 |
| glitch-text | 故障文字 | 文字像信号故障一样撕成红青色条带。最适合醒目标题。 |
| text-on-path | 路径文字 | 文字沿波浪、弧线或圆形行进。最适合徽章和印章。 |
| wavy-text | 波浪文字 | 波浪逐字滚过整个词。最适合俏皮标题。 |
| highlighter-sweep | 荧光笔扫过 | 记号笔依次扫过关键词的后面。最适合要点。 |
| split-flap-board | 翻牌器 | 像机场翻牌一样，翻牌逐个拼出一个词。最适合主视觉标题。 |
| neon-flicker | 霓虹闪烁 | 发光的字母偶尔像破旧霓虹灯一样闪烁。最适合夜晚主题。 |
| handwriting-draw | 手写描绘 | 一个词像用笔写出来一样一笔一画出现。最适合问候语和签名。 |
| text-particles | 文字粒子 | 由圆点组成的词被指针打散，又弹回原位。最适合开场。 |

05 备注：演示示例词保留英文（被演示的文字本身）——kinetic/enter-exit 的短语、typewriter 的打字句（含 " — always." 重打结尾）、rotate-word 的句子与轮换词、morphing 的词（OCEAN 等）、glitch 的 GLITCH、outline 的 OUTLINE、split-flap 的 BOARDING、neon 的 OPEN、gradient 的 Flow、wavy 的 Wavy、on-path 的 FOLLOW THE CURVE ·、scramble 的 DECODE THE MESSAGE、clip-path 的行、particles 的 HELLO、marquee 默认已是中文（动画手册）；handwriting 的词 hello/thanks/hi there（与手绘 SVG 笔画路径绑定）；split-flap 的 A–Z 翻牌。行为适配：highlighter 的演示句已翻成中文（高亮由 `<mark>` 标签驱动，安全）；split-flap 的 "Departures" 牌头翻成“出发”；neon 的 `replace()` 搜索串改为中文从句（否则静默失效）；JS 生成的说明模板全部翻成中文（morphing WORDS/HOLDS/moveWords、wavy `words()`/`secs()`/`NUM` 英文数字词 → 中文数字、on-path `slideWord` 分数 → 中文分数、scramble 速率说明、typewriter aria-label 后缀、handwriting `MORE`、variable `ago()`、gradient "In the box"、particles "Far"、variable "Leans by tilting a little"、wavy `other[]` still 变体、neon `nameFor`）。**hb-cat 行未翻**（`05.NN · Text & Typography`，负责人认为属模板不在其范围；01/02/03/04/06/07 均已翻成中文——协调人汇总时用脚本把 05 的 19 页改为 `05.NN · 文字与排版`）。跨分类 "See also" 保留英文名，待协调人统一改：01 — Text Fill on Scroll、Reveal on Scroll；02 — Clip-Path Reveal（×2）、Curtain Reveal、Letter-by-Letter Stagger、Word-by-Word Reveal、Flip In；04 — Checkmark Draw、Rolling Numbers；06 — SVG Path Animation、3D Flip Card、Canvas Particle Effect、Image Distortion on Hover、GPGPU Particle System；07 — Breathing Glow、Light Leak；分类 README 的 See also：02 — Entrance & Exit、04 — Micro-Interactions。

### 03 页面切换（17）—— 已翻译（2026-11-23，分类负责人已报完；待协调人汇总进 CATS）

| slug | 中文名 | 中文简介 |
|---|---|---|
| view-transitions-api | View Transitions API | 浏览器帮你完成两个页面之间的切换动画。最适合 Web 应用。 |
| shared-element-transition | 共享元素过渡 | 图片从列表放大，进入下一页的页眉。最适合图库。 |
| morph-transition | 变形过渡 | Logo 随你访问的每个页面变换形状。最适合品牌标志。 |
| crossfade | 交叉淡入淡出过渡 | 旧页面淡出，新页面同时淡入。最适合平静的页面切换。 |
| slide-transition | 滑动过渡 | 页面横向滑入，返回时反向滑出。最适合分步流程。 |
| zoom-transition | 缩放过渡 | 页面切换时缩放，仿佛在纵深中移动。最适合打开详情。 |
| flash-transition | 闪光 / 漏光过渡 | 一道闪光遮住页面切换的瞬间。最适合大胆活泼的网站。 |
| blur-transition | 模糊过渡 | 旧页面模糊淡出，新页面逐渐清晰。最适合摄影网站。 |
| elastic-transition | 弹性过渡 | 新页面滑入时冲过头再弹回来。最适合趣味应用。 |
| portal-zoom | 传送门 / 隧道缩放 | 下一页从你点击的圆圈中展开。最适合重大揭示。 |
| dissolve | 溶解过渡 | 页面碎成小块，让位给下一页。最适合图片画廊。 |
| flip-technique | FLIP 技巧 | 布局变化时卡片滑到新位置。最适合排序列表。 |
| circle-reveal | 圆形揭示 | 页面从你点击的任意位置展开，收回时缩回该处。最适合应用导航。 |
| overlay-wipe | 覆盖擦除 | 一块彩色面板扫过页面完成切换。最适合作品集网站。 |
| page-curl | 翻页卷角 | 页面像纸一样从角落卷起。最适合数字杂志。 |
| cube-transition | 立方体过渡 | 页面贴在立方体上，旋转展示下一页。最适合图片故事。 |
| pixelate-transition | 像素化过渡 | 页面碎成大像素，再清晰成下一页。最适合画廊。 |

03 备注：「View Transitions API」保留英文名（API 专名）；「FLIP 技巧」保留 FLIP 缩写。演示内假网站正文、FLIP 卡片名、pixelate 画布文案、shared-element 项目名属演示内容本身，保留英文；导航按钮名、演示内标签/按钮/说明文字已翻译。行为适配（最小改动）：cube-transition 的 `wordsIn.replace('from above','from '+from)` 改为中文（`from` 值改为 `'上方'`/`'侧面'`，replace 目标改为 `'从上方'`）；portal-zoom 的 `NUM` 数组（一~六）与 cube 的 `about()` 秒数词（`['两','三','四']+' 秒'`）。跨分类 "See also" 保留英文名，待协调人用汇总术语表统一改：dissolve → Shatter Effect；flip-technique → Drag to Reorder；circle-reveal → Clip-Path Reveal、Click / Tap Ripple；overlay-wipe → Curtain Reveal、Section Wipe；page-curl → 3D Flip Card；cube-transition → Scroll-Driven 3D Rotation、3D Flip Card；pixelate-transition → ASCII / Halftone。

### 06 3D 与高级效果（28）—— 已翻译（2026-11-23；演示页负责人与 README 补翻负责人均已报告，协调人校验 0 问题）

| slug | 中文名 | 中文简介 |
|---|---|---|
| 2-5d-pseudo-3d | 2.5D / 伪 3D | 平面图层以不同幅度滑动，假装出景深。适合主视觉场景。 |
| 3d-model-orbit | 3D 模型环绕 | 带光照的 3D 形状会自己旋转，或转向你的指针。适合产品展示。 |
| ascii-halftone | ASCII / 半调 | 动态画面用字母或圆点重新画出来。适合复古和科技风格。 |
| audio-visualizer | 音频可视化 | 音条随音乐起舞，从低音到高音。适合音乐播放器。 |
| canvas-particle-effect | Canvas 粒子效果 | 圆点漂浮，靠近时连线，还会躲开指针。适合科技类网站。 |
| carousel-3d | 3D 轮播 | 卡片围成一圈，在 3D 中随拖动旋转。适合图片画廊。 |
| chromatic-aberration | 色差 | 一个词裂成红、绿、蓝三色边缘。适合醒目的标题。 |
| cloth-simulation | 布料模拟 | 一块布在风中飘动，还可以拖着玩。适合有趣的页面。 |
| depth-map-photo | 景深照片 | 平面照片随指尖变成 3D：近处移动得更多。适合主视觉图片。 |
| flip-card-3d | 3D 翻转卡片 | 卡片在 3D 中翻过来，露出背面。适合个人和产品卡片。 |
| flocking | 鸟群 | 鸟儿成群盘旋、一起转向。适合安静的背景。 |
| fluid-simulation | 流体模拟 | 色块像液体一样漂移、融化、合在一起。适合主视觉背景。 |
| glassmorphism-animated | 动态毛玻璃 | 磨砂玻璃卡片模糊了身后的颜色。适合卡片和面板。 |
| gpgpu-particle-system | GPGPU 粒子系统 | 数万个粒子在显卡上流动。适合主视觉特效。 |
| holographic-card | 全息卡片 | 卡片随指尖倾斜，彩虹箔片随之闪烁。适合收藏品。 |
| image-distortion-hover | 悬停图片扭曲 | 图片在指针周围起涟漪、弯曲。适合作品集图片。 |
| infinite-zoom | 无限缩放 | 视野不断放大，进入画中画。适合页面开场。 |
| liquid-glass | 液态玻璃 | 拖动玻璃面板，身后的画面在边缘弯曲。适合卡片。 |
| morphing-blob | 变形色块 | 色块融在一起，其中一滴追着你的指针。适合主视觉区域。 |
| noise-based-motion | 噪声驱动运动 | 平滑噪声让圆点摇摆、色块起伏。适合安静的背景。 |
| parallax-3d-tilt | 视差 3D 倾斜 | 卡片朝指针倾斜，一道光随之滑过。适合卡片。 |
| physics-2d | 2D 物理 | 球从你点击的地方落下，弹跳、堆积。适合有趣的开场。 |
| ray-marching-sdf | 光线步进 / SDF 场景 | 只用公式画出的 3D 场景，镜头绕着它转。适合艺术页面。 |
| scroll-driven-3d-rotation | 滚动驱动 3D 旋转 | 滚动会让 3D 立方体从一个姿态转到下一个。适合产品导览。 |
| svg-path-animation | SVG 路径动画 | 线条画一笔一笔自己画出来。适合图标和标志。 |
| volumetric-smoke | 体积烟雾 / 3D 噪声 | 柔和的烟雾向上盘旋，像真正的 3D 云。适合有氛围的背景。 |
| water-ripple | 水面涟漪 | 涟漪从指尖扩散，弯曲下面的画面。适合主视觉图片。 |
| webgl-shader-animation | WebGL 着色器动画 | 逐像素计算的流动色彩图案。适合大胆的背景。 |

06 备注：演示页术语表由协调人从已翻 h1/lede 提取；README 由补翻负责人翻译，协调人校验 0 问题（28 个 README 标题/章节/参数表/在线演示、分类 README 标题/介绍/28 条列表）。补翻负责人核对：参数默认值以页面实际默认（`class="on"`）为准；正文按钮名用「」标出；两处无链接的同分类引用用了中文名（physics-2d "as Cloth Simulation does"、cloth-simulation "the GPGPU particle demo"）；液态玻璃正文引用的帮助行按页面实际中文文案写。**协调人修了 liquid-glass 演示页 CSS 注释里的一处中英混杂**（`…shadow and 磨砂程度 but no bend…` → `…shadow and frosting but no bend…`，代码注释保持英文）。06 的 README "另见"跨分类英文名（补翻负责人报告，汇总时用 h1 映射脚本统一改）：Parallax Depth-of-Field（01）；Matrix Rain、Grain / Film Noise Overlay、Starfield / Space Particles、Abstract Geometric Motion、Particle Constellation、Flow Field、Game of Life、Mesh Gradient Animation、Ambient Ripple Effect、Ken Burns Effect（07）；Text Particles、Glitch Text、Scramble / Glitch Text、Text Clip-Path Reveal、Outline to Fill、Handwriting Draw（05）；Snap Scrolling、Swipe to Dismiss、Modal Expand、Spotlight Hover Glow、Animated Gradient Border、Hover State Animation、Gooey Menu、Checkmark Draw（04）；Flip In、Bounce In、Page Curl、Portal / Tunnel Zoom、Scrub Animation、Sticky Section、Cube Transition（02/03）；正文里无指向其他动画的链接。分类 README：标题 `# 3D 与高级效果`、介绍、28 条列表（`## 全部动画`）、`## 关键概念`、`## 浏览器要求`、`## 另见` 已翻；“另见”分类名仍英文（04 — Micro-Interactions、05 — Text & Typography）→ 汇总时改中文名。

### 07 氛围与背景（26）—— 已翻译（2026-11-23；负责人未交报告，协调人用脚本校验：0 问题）

| slug | 中文名 | 中文简介 |
|---|---|---|
| abstract-geometric-motion | 抽象几何动效 | 图形旋转、扩散、流动，形成平静无尽的图案。最适合音乐播放器。 |
| ambient-ripple | 环境涟漪 | 涟漪从几个点扩散开来，如静水面上的水滴。最适合主视觉区。 |
| animated-gradient-background | 动态渐变背景 | 颜色在柔和的渐变背景上缓缓流动。最适合主视觉区。 |
| aurora | 极光 | 模糊的光带如北极光般摇曳。最适合深色背景。 |
| breathing-glow | 呼吸光晕 | 柔和的光晕缓缓变大变小，如平静的呼吸。最适合闲置状态。 |
| fire-effect | 火焰效果 | 火焰闪烁升腾，随热量升降。最适合游戏和活动页面。 |
| fireworks | 烟花 | 火箭升空，爆成发光的火花，落下消散。最适合庆祝场合。 |
| floating-elements | 漂浮元素 | 小图形缓缓漂移，各走各的路径。最适合主视觉背景。 |
| flow-field | 流场 | 粒子沿看不见的水流前进，留下渐隐的轨迹。最适合艺术页面。 |
| game-of-life | 生命游戏 | 细胞按邻居生死，图案不断生长。最适合科技类网站。 |
| grain-overlay | 颗粒噪点叠加 | 细腻的颗粒在页面上闪烁，如老电影。最适合编辑类网站。 |
| grid-dot-pattern-parallax | 网格 / 圆点图案视差 | 圆点网格随指针轻轻移动，营造深度感。最适合科技类网站。 |
| ken-burns | 肯·伯恩斯效果 | 照片缓缓缩放平移，再淡入下一张。最适合主视觉幻灯片。 |
| light-leak | 漏光 | 温暖的光线不定时地从角落漫入。最适合照片类网站。 |
| light-rays | 光束 | 柔和的光束从上方洒落，缓缓扫过。最适合主视觉开场。 |
| lightning | 闪电 | 锯齿状的闪电从云层劈下，照亮天空。最适合戏剧性开场。 |
| matrix-rain | 矩阵雨 | 发光的字符列在深色屏幕上落下。最适合科技主题。 |
| mesh-gradient | 网格渐变 | 模糊的彩色色块漂移、交融。最适合落地页。 |
| particle-constellation | 粒子星座 | 漂移的圆点靠近时用线连起来。最适合科技类网站。 |
| plasma | 等离子场 | 平滑的色彩波浪无尽流动，全部由数学生成。最适合创意网站。 |
| scanline | 扫描线效果 | 深色线条和扫过的光带，模仿老电视屏幕。最适合复古网站。 |
| snow-rain | 雪 / 雨 | 雪花飘落，或雨丝落下，洒在黑暗的小镇上。最适合季节性页面。 |
| starfield | 星空 | 星星从黑暗中向你飞来。最适合太空主题。 |
| synthwave-grid | 合成波网格 | 发光的网格在条纹太阳下向你滚动而来。最适合音乐和游戏。 |
| topographic-lines | 等高线 | 如徒步地图上的线条，缓缓弯曲漂移。最适合户外品牌。 |
| wave-layers | 波浪层 | 层层波浪横向漂移，各自速度不同。最适合主视觉区。 |

07 备注：负责人已报告，术语表与报告完全一致，协调人校验 0 问题（JSON-LD、meta/og 描述、提示词结尾、README 标题/章节/参数表、上一页/下一页）。负责人最后补翻了分类 README（标题 `# 氛围与背景`、开头介绍段、`## 动画` / `| 演示 | 描述 |`、氛围思维 / 实现概要 / 另见 全翻）并清掉了 57 处同分类 "另见"英文名（协调人重扫确认：同分类英文仅剩 03 的 View Transitions API / FLIP 技巧与 01 的“滚动 scrub 动画”共 21 处，均为中文名本身含拉丁字母，不是问题）。行为适配：animated-gradient-background 的 `s.replace('four','three')` 改为 `replace('四','三')`，inside/still 说明措辞保证恰好一个“四”；light-leak 的 `words.inside.replace(...)` 搜索串改为中文对应句；CORNERS 值 → 左上/右上/左下/右下。07 的 README "另见"跨分类英文名（负责人报告与协调人 grep 一致，汇总时用 h1 映射脚本统一改）：WebGL Shader Animation、Noise-Based Motion、Audio Visualizer、Click / Tap Ripple、Water Ripple、Cloth / Soft-Body Simulation、Volumetric Smoke / 3D Noise、Success Confetti、Canvas Particle Effect、Flocking、Chromatic Aberration、2.5D / Pseudo-3D、Parallax 3D Tilt、Infinite Zoom、Zoom Into Image、Glassmorphism Animated、Flash / Light Leak Transition、ASCII / Halftone、Marquee / Ticker、Parallax Scrolling。分类 README 待汇总规范化：条目为表格（`## 动画` + `| 演示 | 描述 |`）→ 转为列表并改标题为 `## 全部动画`；`## 另见` 的分类名仍英文（06 — 3D & Advanced、04 — Micro-Interactions）→ 改中文名。

## 刻意保留英文的内容

- 动画 slug、文件/文件夹名、URL、CSS 属性与值、class 名、JS 标识符。
- 代码块、代码注释里的标识符；`docs/superpowers/` 未翻译（开发设计文档）。
- shields.io 徽章 URL（`badge/techniques-176-…`）。
- 演示页内嵌的演示文案中属于演示内容本身的英文单词（如演示里展示的英文单词本身就是演示对象时，译员判断；一般仍翻译，除非它是被演示的文本内容）。
- `data-hb-motion-note` 的值不含英文（中文句子）。

## 进度

- [x] 术语表与规则（本文件）
- [x] 共享界面机械替换（176 页模板文案 + demo-page.js + demo-page.css 注释 + 404.html + 测试同步；`node --test "tests/*.test.js"` 2371/2371 通过）
- [x] 首页（根 index.html）UI 翻译：head/title/JSON-LD、导航、hero、PLACES、FIND_Q、结果区、知名效果、怎么用、页脚、noscript、render/findLine/showFound/showFinder/findWords；搜索分词 CJK 化（`wordsOf`：英文词 + 中文词组 + 单字；STOP 增加中文虚词）；`tools/check-pages.mjs` 的首页/演示页断言同步为中文；`tests/pages.test.js`、`tests/finder.test.js` 同步。首页测试全过；其余失败全部是分类页与未翻译 CATS 之间的逐页一致性检查，待 CATS 汇总后消除。
- [x] 01 滚动驱动（27）—— 翻译完成（负责人未交报告，协调人从文件校验），术语表已汇总在本文件，待协调人写入 CATS
- [x] 02 入场与退场（15）—— 翻译完成，术语表已汇总在本文件，待协调人写入 CATS
- [x] 03 页面切换（17）—— 翻译完成，术语表已汇总在本文件，待协调人写入 CATS
- [x] 04 微交互（44）—— 翻译完成（负责人已报告，协调人校验 0 问题；修了 2 处脚本语法错误），术语表已汇总在本文件，待协调人写入 CATS
- [x] 05 文字与排版（19）—— 翻译完成，术语表已汇总在本文件，待协调人写入 CATS
- [x] 06 3D 与高级效果（28）—— 翻译完成（演示页 + README 负责人均已报告，协调人校验 0 问题），术语表已汇总在本文件，待协调人写入 CATS
- [x] 07 氛围与背景（26）—— 翻译完成（负责人已报告，协调人校验 0 问题），术语表已汇总在本文件，待协调人写入 CATS
- [x] 根 index.html（CATS + 搜索 placeholder + Popular 搜索词 + 分组 chips 名称）—— 已翻译并提交（2a09a3e）：CATS 176 条中文名/简介 + `key` 字段（英文 slug，解 `?cat=` 冲突）；搜索分词已 CJK 化；`tests/pages.test.js`、`tests/finder.test.js` 的搜索期望改中文并实测
- [x] 根 README.md —— 已翻译并提交（0984cd4）：正文 + 完整目录（176 条由 CATS 脚本重新生成；注意 `cat.folder` 无尾斜杠，拼接须加 `/`）；`tools/update-sitemap.mjs` 的 llms.txt 硬编码文案已翻中文并重新生成（llms.txt 176 页、sitemap.xml 178 URL）
- [x] llms.txt / sitemap.xml 重新生成 —— 已完成（0984cd4）；**注意**：famous 页 meta 描述已改中文后 llms.txt 的知名效果行会变旧，famous 全部做完后须再跑一次 `node tools/update-sitemap.mjs`
- [x] famous/ 首页 —— 已翻译并提交（4958350）：11 个条目中文标题、tile 一行描述、what、know 三行、提示词（中文提示词，以“给”开头）、卡片名/简介（取自 CATS）、标签；`tools/make-famous.mjs` 已重跑（23 卡片、22 预览）；`tests/famous.test.js` 与 `tools/check-pages.mjs` 的 famous 断言同步为中文；`tests/famous.test.js` 除 11 个版本页标题检查外全过
- [x] famous/ 11 个版本页（famous/versions/*.html）—— 已翻译并提交（子代理 9445ad4c，协调人校验）：标题 `我们的版本：<中文标题>` 与 famous 首页 h2 逐字一致；演示文案翻中文（菜单、歌单、文章、步骤计数 第 N 步，共 4 步、按钮、aria-label）；JS 生成文案两侧同步翻译（spotify `data-t` 与可见名、`播放<名>` aria-label；wikipedia `CARDS` 数组与 `已打开链接：`；youtube `CARDS` 标题/频道与 `正在加载视频 / 视频已加载`；pudding `SAY` 与分组标签）；`data-fx-touch="drag"` 仅 landonorris-reveal 有；noindex 保留；可见文案均 > 40 字、无站点名与 SITE_PHRASES（协调人独立重扫确认；唯一英文词是 wikipedia aria-label 里的按键名 `Esc`，按键名按规则保留英文）；youtube-loading 因 JS 建网格、静态文案不足 40 字，测试曾失败一次，子代理把网格 aria-label 与初始状态文案加长后通过
- [x] 测试全过 —— `node --test "tests/*.test.js"` 2371/2371 通过；`node tools/update-sitemap.mjs` 已重跑（llms.txt 知名效果行改为中文描述；sitemap.xml 无变化）
- [ ] 页面检查（Chrome 在本机无法启动，见下）

## famous 翻译（2026-11-23，协调人；版本页子代理进行中）

- famous/index.html：标题 `知名效果 — 动画手册`；lede/法律行/hero pill（`11 个来自知名网站的效果`）/导航/页脚/弹窗按钮（`上一个效果`/`下一个效果`/`关闭`，均为 aria-label）/dnote 全部翻成中文；11 个条目：h2 中文标题、tile 一行描述（25–27 字，以 。 结尾）、what、know 三行（开始方式/在手机上/减弱动态效果）、提示词（中文，以“给”开头，`[部分]` 包在 `<mark>` 里）、卡片名/简介取自 CATS（脚本按 slug 查表替换）、卡片标签（2–6 字短标签）。
- 11 个中文标题：扭动的光带 / 随滚动拆开的头显 / 指针下的液态揭示 / 由模糊变清晰的标题 / 一个接一个落下的菜单 / 链接预览 / 扫过标题的色彩 / 升起来的播放按钮 / 加载时的灰色形状 / 读着读着就变的图表 / 读文章时藏起来的顶栏。
- Seen on 行格式：`见于 <a>host</a>，<time>2026</time>&nbsp;年`（年份与“年”之间不换行空格；原 `in&nbsp;` 结构已无对应物）。CSS 里 `.fx-seen` 的注释已按新格式改写。
- 复制按钮：静态 aria-label `复制「标题」的提示词`；`copyState` 三种 aria-label（`已复制「标题」的提示词` / `按 ⌘C 复制：「标题」的提示词已选定` / `复制「标题」的提示词`）；播放器按钮 暂停/播放/重播/演示一下/回到顶部；提示（fx-phint）按模式翻成中文（滚动提示原文测试固定为 `按下面的提示词做的。在框里滚动，或者点「播放」。`）。
- 测试计数改为 CJK 感知：`words()` = 英文词数 + 中文字符数；what 50–120（实测 63–102）、提示词 140–220（实测 158–199）、know 行 2–35（实测 8–27）、卡片标签 1–6 字（实测 2–6）、tile 行 25–90 字（实测 25–27，注意：手工数字数易少 1–2 字，须以脚本实测为准，如 spotify 的 tile 行初版 24 字）。
- `stems()` 改为英文 5 字母词干 + 中文双字（bigram）；COMMON_STEMS 实测 12 个，全来自减弱动态效果句（如果/访客/开启/了减/减弱/弱动/动态/态效/效果等）；know 行与各自提示词共享非公共词干最少 2 个（实测最低 2 个：apple-scroll 与 landonorris 的第一行），改词时须重测。
- 提示词规则：以“给”开头；含减弱动态效果句；hover/press 条目须含 触摸/轻点/手指 词；不得出现站点名与 SITE_PHRASES；landonorris 的提示词须在同一句里同时出现 手指 与 拖动（`data-fx-touch="drag"` 的测试要求）。
- 卡片 chips：`由 N 个动画组成`（测试模板是 `` `由 ${animations(n)}组成` ``，注意 `组成` 前无空格，动画数词后直接接 组成；英文原模板无此问题）。
- `tools/make-famous.mjs` 已重跑（23 卡片、22 预览写入）；`tools/check-pages.mjs` 的 famous 断言已同步（暂停/播放、复制提示词/已复制/按 Ctrl+C 复制、下一个效果/上一个效果、toast 中文文案）。
- famous 页 CSS 注释保持英文（开发用描述，不可见）；只有 `.fx-seen` 的换行注释按新格式改写。JSON-LD：famous 页没有 JSON-LD 块。
- 故意保留英文：`data-fx-site` 属性值（Stripe 等站点名，测试用它派生 siteWords）、seen 链接的 host 文本（stripe.com 等地址本身）、`.github/ISSUE_TEMPLATE/famous_effect.yml`（GitHub 侧表单，测试固定英文）。

### famous 版本页（子代理进行中）

- 11 个文件 `famous/versions/*.html`：`<title>` 改为 `我们的版本：<中文标题>`（标题同上），演示文案翻中文；约束：noindex 保留；无外部引用（仅 `../../assets/fonts/`、`data:`、`#`）；脚本保留 matchMedia 减弱动态检查 + `@media (prefers-reduced-motion: reduce)` 规则 + 只信任 parent 的 message 监听；`FRAME = 1000/60`、`Math.min(ts - lastT, 50)`、`dt / FRAME` 模式不动；滚动版保留 `overscroll-behavior: contain`，非滚动版不得有；`data-fx-touch="drag"` 仅 landonorris-reveal 有；可见文案 > 40 字且不得含站点名与 SITE_PHRASES；spotify-cards 的 `data-t` 属性与可见名、aria-label 须同步翻译。

## 首页翻译暂缓项（CATS 汇总时一起做）

- 搜索框 placeholder（`例如：点击时会弹跳的按钮`）：`tests/pages.test.js:1090` 会读它去搜并期望 Button Press Scale 首位、Bounce In 在第一页——须等 CATS 翻译后，把 placeholder 换成中文例句并把期望改成中文名。
- Popular 搜索词（hero 与空面板的 5 个 `data-q` + 可见词）：须等 CATS 翻译后换成中文查询并验证首位结果；`tests/pages.test.js` 的 `popular` 映射同步改中文。
- 分组 chips 的 7 个分类名（spots 区）：须等 CATS 翻译后改为 `cat.name` 中文名。
- CATS 本身未动：`name`/`desc` 仍是英文；plain-links（noscript 列表）与 llms.txt 仍是英文，随 sitemap 重新生成一起更新。

## 首页搜索行为变化（已验证）

- `placeW` 现在是中文位置名（按钮/文字/图片与卡片/背景/菜单与表单/加载与提示/页面开场/滚动/页面切换），英文查询 `page change animation` 不再首位返回页面切换卡片——测试已临时改为查询 `page transitions`（仍首位返回页面切换卡片，期望分类名仍为英文 `Page Transitions`）；CATS 汇总时把该行改为中文查询 + 中文分类名 `页面切换`。
- 中文查询目前只能通过位置名匹配（如 `页面切换动画` 能找到页面切换位置的卡片，但首位按 home 顺序，未必是页面切换分类）；`淡入` 目前返回空——CATS 翻译后中文名/简介进索引即恢复。不要为此加中文 SAME 组（会动 `tests/pages.test.js` 的 SAME 遍历测试）；CATS 汇总时再实测决定。
- `wordsOf` 不再用 `[^a-z0-9]+` 切分，改为 `matchAll(/[a-z0-9]+|[\u4e00-\u9fff]+/g)`：英文行为不变；中文词组同时产生整词与单字 token。`WORDS` 保持英文（测试要求 `/^[a-z0-9]+$/`）。

## 待协调人统一做的事（不要由分类负责人改）

- **CATS 的 `key` 字段**：`const CAT=Object.fromEntries(CATS.map(c=>[slugify(c.name),c]))` 用分类名做 `?cat=` 的 key——中文名经 `slugify` 全变空串，7 个分类冲突。CATS 每项加 `key` 字段（英文 slug：scroll-based、entrance-exit、page-transitions、micro-interactions、text-typography、3d-advanced、ambient-background），CAT 行改为用 `c.key`；同步改 `tests/pages.test.js` 的 chips 测试（`key: cat.key`）、`?cat=` 测试与 `homeAddress`（不再取 `const slugify=` 行）、spots 的 `data-cat` 不用动（已是英文 slug）。
- **README 动画名链接统一改中文（汇总脚本，06 报告后再跑）**：各分类 README 的 "另见"与正文里指向其他动画的链接，链接文字仍是英文原名。汇总时用脚本按 href 目标 slug 查 h1 中文名统一替换（只改 href 指向 `animations/<分类>/<slug>/` 的链接，不动外部链接与代码块；目标 h1 本身就是英文的如 View Transitions API、FLIP 技巧、滚动 scrub 动画，替换后不变）。协调人已用 `/tmp/seealso.cjs` 扫过全库（以 `## 另见` 分段 + href 解析为准；注意：以数字开头的原名如 3D Flip Card、2D Physics，`/^[A-Za-z]/` 会漏检，要用 `/[A-Za-z]/`）："另见"跨分类英文 82 处、同分类英文 57 处（全在 07；03 的 8 处 View Transitions API / FLIP 技巧与 01 的 13 处“滚动 scrub 动画”是中文名本身含拉丁字母，不是问题）；正文里的英文动画名链接主要在 06 的 README（28 页，负责人翻译 README 时可能已处理，汇总时以脚本扫出的为准再改）与 05/text-clip-path-reveal（Reveal on Scroll）、07/ambient-ripple（Click / Tap Ripple）、07/scanline（Grain Overlay、Chromatic Aberration）。
- **分类 README 规范化（汇总时统一）**：标题——01/02/03/05/07 为 `# 中文名`，04 为 `# 04 — 微交互` → `# 微交互`，06 仍英文（负责人进行中）；条目区——01/02/03/05 为列表，04/07 为表格 → 统一转为 `- [中文名](slug/) — 中文简介` 列表，标题统一为 `## 全部动画`（现状：01/03 `## Entries`、02 `## 全部动画`、04/05 `## 动画`、07 `## Animations`）；07 的开头介绍段、`## 实现概要` 表、`## 另见` 描述仍英文 → 翻译；04 的 `## 关键概念` 已翻译，`## 另见` 分类名改中文。
- **分类间不一致，汇总时统一**：① `hb-cat` 行——01/02/03/04/06/07 已翻成中文（`NN.NN · 中文分类名`），只有 05 的 19 页仍是英文 `05.NN · Text & Typography`，汇总时用脚本改掉。② 分类 README 的条目区标题——02 用了 `## 全部动画`、05 用了 `## 动画`、01/03 仍是 `## Entries`、04/06/07 仍是 `## Animations`，汇总时统一为 `## 全部动画`。
- **搜索测试改中文**（`tests/pages.test.js:1086` 起与 `tests/finder.test.js` 的 CASES 28 条）：placeholder 例句、`first` 映射、parallax 断言、`page transitions` 行、Popular 映射、CASES 的英文卡片名，全部改为中文查询 + 中文期望名；`parallax` 的 SAME 组如需中文对应（视差/深度）须先实测 `tests/pages.test.js:1120` 的 SAME 遍历（直接匹配须排在组匹配前面）。
- **famous**：CATS 翻译汇总后再跑 `node tools/make-famous.mjs` 重新生成 famous 页的生成块。
- **check-pages 的 famous 部分未动**：`tools/check-pages.mjs` 里 famous 页的断言仍是英文（Pause/Play、Copy prompt/Copied、Press Ctrl+C to copy、Next/Previous），随 famous 翻译一起改。
- **llms.txt 的 Famous effects 标题**：`tests/pages.test.js:917` 固定 `'## Famous effects'`——跑 `update-sitemap.mjs` 翻译其硬编码串时同步改测试。
- **页面检查跑不通（环境问题，待父代理另行安排）**：本机 `/opt/meta-chromium/chrome` 启动即报错 `Hatch WebGL domain policy is required for browser startup`（直接启动与 Playwright 启动均失败，与翻译无关）；`node tools/check-pages.mjs home` 因此无法运行，首页只做了单元测试 + 脚本编译 + JSON-LD 校验。须在能启动 Chrome 的环境补跑 `home` 与抽查分类。
- **README 章节标题翻转**：各分类把动画 README 的章节标题翻成中文（它是什么 / 什么时候用 / 工作原理 / 关键参数 / 生产注意事项 / 另见）后，协调人把 `assets/js/demo-page.js` 里 `loadReadme()` 的 `s['What it is']` / `s['See also']` 改成中文 key，同时改 `tests/pages.test.js` 的 `keyParameters()`、`sections()` 查找与章节要求列表，并翻译 demo-page.js 文件头注释（第 1–13 行）里提到的章节名。
- **motion-note 正则**：`tests/pages.test.js:143` 的 `/^[A-Z][^.]*[^.\s]$/` 在 01 分类把 `data-hb-motion-note` 翻成中文后，改成中文规则 `/^[^。.]*[^。.\s]$/`（一句中文，无句号，结尾不是标点或空白）。
- **首页与根 README**：CATS 翻译汇总后由协调人统一用脚本写入根 index.html，再跑 `node tools/update-sitemap.mjs` 重新生成 llms.txt、sitemap.xml 与首页 no-JS plain-links；`tools/update-sitemap.mjs` 里的 llms.txt 开头硬编码英文要先翻成中文。
- **famous**：CATS 翻译汇总后再跑 `node tools/make-famous.mjs` 重新生成 famous 页的生成块。
