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
- 条目格式不变：`- **[中文名](https://matinmonshizadeh.github.io/animation-handbook/animations/…/)** — 中文简介`，简介与 CATS 的 desc 完全一致。
- `All 176 techniques, each linked to its live demo.` → `全部 176 个动画，每个都有在线演示链接。`
- Categories 表：分类名用中文，链接不变，计数不变。

## 工具脚本里的硬编码英文

- `tools/update-sitemap.mjs` 的 llms.txt 开头文案是硬编码英文（`# Animation Handbook`、`> A visual reference of…`、`Each page plays one technique live…`、`## Famous effects`、`## Optional`、`- [Source on GitHub]…`），改工具本身的文案再重新生成，否则重新生成会把 llms.txt 翻回英文。
- `tools/make-famous.mjs` 从首页重新生成 famous 页的生成块（PREVIEWS 与预览 CSS），**在 CATS 翻译完成后再运行**。
- `tools/update-sitemap.mjs` 重新生成 llms.txt、sitemap.xml 与首页 no-JS  plain-links，**在 CATS 翻译完成后再运行**。

## 动画术语表（176 个动画的中文名 + 一行中文简介）

由各分类负责人起草、协调人汇总后填入本节。规则：中文名要自然、符合中文前端开发者的习惯说法；简介是 "Best for …" 句的完整翻译；slug 保持英文；简介在 CATS、演示页、README、分类 README、根 README 五处完全一致。

### 01 滚动驱动（27）

（待填）

### 02 入场与退场（15）

（待填）

### 03 页面切换（17）

（待填）

### 04 微交互（44）

（待填）

### 05 文字与排版（19）

（待填）

### 06 3D 与高级效果（28）

（待填）

### 07 氛围与背景（26）

（待填）

## 刻意保留英文的内容

- 动画 slug、文件/文件夹名、URL、CSS 属性与值、class 名、JS 标识符。
- 代码块、代码注释里的标识符；`docs/superpowers/` 未翻译（开发设计文档）。
- shields.io 徽章 URL（`badge/techniques-176-…`）。
- 演示页内嵌的演示文案中属于演示内容本身的英文单词（如演示里展示的英文单词本身就是演示对象时，译员判断；一般仍翻译，除非它是被演示的文本内容）。
- `data-hb-motion-note` 的值不含英文（中文句子）。

## 进度

- [x] 术语表与规则（本文件）
- [x] 共享界面机械替换（176 页模板文案 + demo-page.js + demo-page.css 注释 + 404.html + 测试同步；`node --test "tests/*.test.js"` 2371/2371 通过）
- [ ] 01 滚动驱动（27）
- [ ] 02 入场与退场（15）
- [ ] 03 页面切换（17）
- [ ] 04 微交互（44）
- [ ] 05 文字与排版（19）
- [ ] 06 3D 与高级效果（28）
- [ ] 07 氛围与背景（26）
- [ ] 根 index.html（CATS + 首页文案 + 搜索分词）
- [ ] 根 README.md
- [ ] llms.txt / sitemap.xml 重新生成
- [ ] famous/（首页 + 11 个版本页）
- [ ] 测试全过 + 页面检查

## 待协调人统一做的事（不要由分类负责人改）

- **README 章节标题翻转**：各分类把动画 README 的章节标题翻成中文（它是什么 / 什么时候用 / 工作原理 / 关键参数 / 生产注意事项 / 另见）后，协调人把 `assets/js/demo-page.js` 里 `loadReadme()` 的 `s['What it is']` / `s['See also']` 改成中文 key，同时改 `tests/pages.test.js` 的 `keyParameters()`、`sections()` 查找与章节要求列表，并翻译 demo-page.js 文件头注释（第 1–13 行）里提到的章节名。
- **motion-note 正则**：`tests/pages.test.js:143` 的 `/^[A-Z][^.]*[^.\s]$/` 在 01 分类把 `data-hb-motion-note` 翻成中文后，改成中文规则 `/^[^。.]*[^。.\s]$/`（一句中文，无句号，结尾不是标点或空白）。
- **首页与根 README**：CATS 翻译汇总后由协调人统一用脚本写入根 index.html，再跑 `node tools/update-sitemap.mjs` 重新生成 llms.txt、sitemap.xml 与首页 no-JS plain-links；`tools/update-sitemap.mjs` 里的 llms.txt 开头硬编码英文要先翻成中文。
- **famous**：CATS 翻译汇总后再跑 `node tools/make-famous.mjs` 重新生成 famous 页的生成块。
