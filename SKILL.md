---
name: maison-des-mots-art-direction
description: 设计并实现「法式洛可可纸雕 / vintage scrapbook」网页 UI。适用于奶白与银蓝配色、模切白边、叠纸信笺、吊牌、手绘线稿贴纸、打字机档案标签、法文花体、GSAP 逐项入场与拼贴交互；也适用于把任何工具类页面改造成「一屏一页、像纸册一样翻」的纸上作品。
license: MIT
---

# Maison des Mots · 法式纸雕网页 Art Direction

一套已经落地验证过的美术方向与实现配方。目标不是「加点复古元素」，而是让整页像一件用纸、线和阴影做出来的实物：**克制的颜色、真实纸感、清晰的层级，以及可以触摸的交互。**

参考实现：`https://github.com/BaxterL/maison-des-mots`（React 19 + Vite + GSAP）。
本文件可整份交给 AI 编码助手，按第 10 节改一句话即可开工。

**一句话原则：单屏、单页、一个平面。** 不靠整页滚动，内容多的时候用「册子 / 便签 / 抽屉」式展开，换页靠逐项入场而不是转场特效。

---

## 1. 先定方向

一句话：**把一间凡尔赛宫的纸艺工作室，装进一个安静的浏览器窗口。**

- 是「纸雕 + 档案检索 + 手账拼贴」，不是泛法式复古，也不是 AI 工作台。
- 高级感来自留白、1px 线、错位叠纸和一致的模切白边，不来自花纹、金边、深色底。
- 每屏只允许一个视觉主角；装饰必须能被忽略，不能被误认为控件。
- 中文负责叙事，法文负责气质，英文衬线负责标题，打字机字体负责编号与档案注脚。
- 每页信息必须在一屏内看完；放不下的内容做成可以点开的「册子」，**不要**把页面拉长。

**不要**：暖金、大面积深色底（文字会糊）、密集波点/网格/噪点、粗野硬阴影、持续旋转的装饰、满屏手写体。

## 2. 设计 Token

```css
:root {
  --ink: #1e2748;          /* 墨蓝，所有文字与线 */
  --ink-soft: #46536f;     /* 次级正文 */
  --fog: #5c6884;          /* 注释、编号 */
  --fog-light: #a9b3cc;    /* 更弱的装饰线 */
  --silver: #bfc8e0;       /* 银蓝，阴影/描边 */
  --silver-light: #dce2f1; /* 雾蓝纸 */
  --paper: #fbf9f4;        /* 奶白底 */
  --paper-light: #fffdf8;  /* 信笺纸 */
  --paper-deep: #eef0f5;   /* 画布底 */
  --lotus: #e8c9d0;        /* 藕粉，只用一点点 */
  --line: rgba(30, 39, 72, .42);
  --hairline: rgba(30, 39, 72, .16);
}
```

**字体**（全部 SIL OFL，本地化，不走 CDN）：

- 标题/英文衬线：`Cormorant Garamond` 400–600 + italic
- 中文标题：`Noto Serif SC` 400–700
- 信笺正文 / 灵感卡片：`LXGW WenKai`（霞鹜文楷）500 / 700
- 法文花体：`Pinyon Script`（一句即可，别超过 2 处）
- 档案/编号/注脚：`Special Elite`
- 可爱纸面（可选，给创作台的字迹选择）：`ZCOOL KuaiLe`（快乐体）、`ZCOOL QingKe HuangYou`（黄油体）

> **分层原则**：WenKai 只给“信/手写”的内容，表单、计数、状态、操作标签继续用清晰的宋体；不要让整个界面都变软。

> **字体本地化**：拉丁字体直接 woff2；中文用 `fontTools.subset` 子集化。
> 站内固定文案按实际用字子集（约 350 字，几十 KB）；
> 需要接住用户输入的字迹字体，按 GB2312 一级常用字 3755 + 拉丁/标点子集
> （本项目：文楷 100KB、思源宋体 217KB、快乐体 452KB、黄油体 609KB）。
> `@font-face` 只在文本真正用到该 family 时才下载，所以可爱字体不影响首屏。
> 生僻字缺字时浏览器自动回退，可以接受。

**纸的物理感**：所有卡片共用一套阴影，不要发明第二种。

```css
--paper-shadow:
  0 0 0 3px rgba(255,255,255,.88),   /* 模切白边 */
  0 0 0 4px rgba(30,39,72,.12),      /* 细墨蓝外线 */
  0 16px 30px rgba(30,39,72,.14);    /* 短距离柔阴影 */
```

**比例**：标题字重 500、行高 1.24–1.45、字距 -0.055em（中文大字）到 +0.06em（正文）。
正文 14–16px / 行高 1.9–2；表单与状态文字 11–13px（**必要信息不要低于 11px**）；微型注脚不低于 9px，且在浅底上保持 `--ink-soft`。
一屏布局里最容易犯的错是「为了塞下而缩字号」：先减装饰与留白，字号最后才动。

## 3. 页面骨架（一屏一本）

```
app
├─ page-texture（全屏 grain，2%）
├─ book-shell（高度 calc(100dvh - 44px)，flex column，最大宽 1680）
│  └─ inner-frame（内缩 15，flex column，min-height:0）
│     ├─ 四角 SVG 花角（aria-hidden，绝对定位）
│     ├─ masthead（品牌 / 当前章节 / 版次，88px）
│     ├─ menu-ribbons（右缘 4 枚三层纸签；≤1080 底部 tab）
│     ├─ main.page-stage（flex:1，min-height:0，overflow:hidden）
│     │  └─ .page-view[hidden] × 4（懒挂载，保留状态，height:100%）
│     │     ├─ 01 couverture：hero + hero-ledger
│     │     ├─ 02 atelier：木桌/亚麻桌 + 表单 + 纸 + 贴纸簿
│     │     ├─ 03 inspiration：3 张示例纸 + 贴纸谱
│     │     └─ 04 maison：居中的“纸房间”宣言
│     └─ footer（3 段小字，28–34px，≤760 隐藏）
└─ （没有 PageTurn：换页是内容逐项入场，不是 3D 翻纸）
```

- `.book-shell { height: calc(100dvh - 44px); display:flex; flex-direction:column }`；
  `.inner-frame`、`main.page-stage` 一路 `flex` + `min-height:0`，这样四页永远在一屏内。
- 每页根节点 `height: 100%`；页面内部各区块用 `flex:1 / min-height:0` 分配，超长内容只在「纸内」滚动（隐藏滚动条），**整页不滚**。
- 路由用 hash（`#/atelier`）；`popstate` 同步；首次无 hash 用 `replaceState` 补上。
- 背景用 3 层低对比 radial-gradient 模拟云石晕染（银蓝 + 一点藕粉），不要用图片。
- 纸张颗粒：全屏 `<svg>` + `feTurbulence`，透明度 ~2%，`mix-blend-mode: multiply`。
- ≤760 手机：`book-shell` 高度 `calc(100dvh - 12px)`，内缩 8px；底部 tab 改成 `position:absolute`（在 `.inner-frame` 内），并给 `.inner-frame { padding-bottom: 68px }`，不要用 body padding-bottom（会产生整页滚动）。

## 4. 首屏配方

**吊牌栏**（桌面 >1280）：左侧 92px 宽、绝对定位，一条银蓝渐变细绳从上挂到底；
5 枚模切纸签（香水瓶、缎带、玫瑰、钥匙、月亮），宽 64 / 高 82，纸签带顶部圆孔和 1px 内框，各自 rotate(-4°…4°)。整条 `aria-hidden="true"`、`pointer-events: none`。

**拼贴**：从后到前是

1. 档案表盘（540px 圆，虚线环 + 48 根刻度 + `textPath` 环形法文，opacity .55）
2. 雾蓝纸（69% × 78%，斜切 clip-path，rotate 5°）
3. 奶油信笺（`min(68%, 430px)`，内缩 9px hairline，rotate -1.5°）
4. 邮票、山茶花吊牌（矢量 SVG 模切线稿）、挂着的香水瓶、Souvenir 菱形卡

> 桌面首屏最多 10 个装饰件；超过就满。装饰只放在拼贴四周，不要压住正文与主按钮。

信笺内部：`LETTRE N° 001 / PARIS · 2026` → 法文斜体小句 → 中文两行 → Pinyon 手写体 → 11px 正文 → 拱廊版画（`mix-blend-mode: multiply`, opacity .82）→ 页脚。

**主标题**：两行中文 `clamp(44px, 4.5vw, 68px)`，每行放进 `overflow:hidden` 的 mask，入场时 `yPercent: 115 → 0`。花体 `Les mots doux` 放标题之后独立一行、右对齐、rotate -6°，**不要**压在主标题上。

**首屏压缩**（避坑）：矮屏（≤920 高）把 hero 内的 `description / actions / proof` 间距一起收紧，
并给 `.hero-proof`（01/02/03 那行）留 `padding-top`，否则它会和下面的 `hero-ledger` 横线卡在一起。
本次实测：1440×900 下 proof 底 657、hero 底 709，安全。

## 5. 书签菜单与「换页」

**不要做传统横条导航。** 四个目的地对应右缘四枚竖排书签：

- 三层纸签束：`ribbon-back / mid / front` 三层错位（各偏 2–6px、略小），`mix-blend` 无、`drop-shadow` 轻；
  宽 48（front）/ 50 / 52，高 106–110，底部斜切，顶部圆孔 + 1px `--lotus` 小点表示 active。
- 文案：中文主标签 12px 竖排居中，编号 9px 退后；active 向左抽出 8px 并换成墨色面。
- 屏幕高度 < 940px 时整体 `scale(.8)`；≤1080 改成底部纸质 tab 条（高 56，绝对定位，`inner-frame` 预留 68–78px）。
- `aria-current="page"`、`aria-label` 带编号与中法文；hover 只位移，不叠额外动效。

**换页（无翻纸）**：

```jsx
// App.jsx
const [entered, setEntered] = useState(true);

const navigate = (id) => {
  setEntered(false);                 // 1. 先把新页的入场打回起点
  setPage(id);                       // 2. 同一帧直接换页（无转场）
  history.pushState(..., `#/${id}`);
  requestAnimationFrame(() => setEntered(true)); // 3. 下一帧开始逐项入场
};
```

- 每个页面组件接收 `enter`：用 `useGSAP` 构建时间线后，`enter ? timeline.play(0) : timeline.pause(0)`，
  页面**挂载时就处在入场起点**（from 的 immediateRender 让元素先隐去）。这样新页永远不会「先完整出现再淡入」。
- 页面组件依赖写成 `[enter]`（不是 `[active]`），保证每次到达都重放，包括从菜单回到封面。
- `prefers-reduced-motion` 下不建时间线，直接就位。
- 换页时先设新页、滚动到顶部（`window.scrollTo(0,0)`），再在 rAF 里放开入场；`main` 拿焦点。
- **不要**再做 3D 翻纸（PageTurn）：纸册一开始确实好看，但 3D 层在合成器上容易闪第一帧，且翻完整页的视觉重量过大。

## 6. 创作台与笔写作

**桌面**（这一页是「在桌子上写」）：整个 stage 是一块浅银灰亚麻桌布 + 极淡 `feTurbulence` 木纹/布纹，
四周一点内阴影、顶部一条银线；纸、信封、墨瓶、便签都压在桌面上。**不要**用暖木色/深色桌——和银蓝纸色打架，字也看不清。

- 左表单：奶白叠纸 + 7px 藕粉纸胶带，字段用 `01 / 02` 档案编号，textarea 带 30px 横线背景；`flex` 布局，超高时只让面板内滚。
- 右画布：`flex:1`，四角 L 形边框，中央信笺 `width min(66%, 520px)`、高度 `calc(100% - 92px)`。
- **信封**：折痕、邮票、火漆印都用 SVG 填充（不要只用描边线，会像线框）；压在信纸左下方，`opacity .9`。
- **墨瓶与毛笔**：右下角，墨瓶点击切换字迹并触发蘸墨动画；毛笔（`.brush` 44px）跟随正文书写位置。
- **双模式**：
  - `用灵感起稿`：主题 + 语气 → 模板补全成一封信；
  - `写自己的`：标题可选、正文必填 → 原样上纸，**绝不追加任何模板句子**。
  - 「编辑这段」把当前纸面文字带回编辑区；空正文在字段附近报错，不用 toast。
- **纸面字体**：文楷 / 宋体 / 拉丁 / 快乐 / 黄油，切换时毛笔蘸墨、状态栏说明。
  CSS 用 `.collage-board.font-xxx .generated-body, .collage-board.font-xxx .generated-title` 覆盖；
  导出时用同一张字体表，否则导出的字和屏幕不一致。
- **书写速度**：慢 / 常 / 快（0.7 / 1 / 1.6 倍），作用在时间线的 `delay / step / 每次 tween 时长` 上。
- **笔写作**：
  - 正文按 2–3 个 token 分组，标点/换行作为停顿；组内 opacity + `blur(1.8px)→0` + `y:1.6→0` 显影。
  - 毛笔按组移动，带轻微 `rotation` 交替（±5°）和 `scaleY`，别让它像光标。
  - 标题不做逐字，先落标题再写正文；写完笔淡出；提供「跳过书写」，切页/编辑时取消时间线。
- **贴纸簿**（一屏内收纳贴纸的做法）：桌角一枚手裁纸册（不规则 `clip-path` + 三层错位纸 + 回形针 SVG + 和纸胶带），
  点开是一张不规则便签页：**便签可按住页眉拖动**；十枚贴纸「点一下放上纸」或「直接拖到纸上」。
  拖拽用 `pointerdown` + 全屏 ghost，`pointerup` 命中画布矩形才落纸。
- 贴纸：10 类手绘线稿（统一 `1.1` 描边、墨蓝+银蓝两色），按钮 92×110，`filter: drop-shadow(...)`；
  选中态只加虚线边 + 极少量藕粉，不加粗、不换尺寸（避免 Draggable 坐标错位）。
- 画布上的贴纸 `position:absolute; inset:0 auto auto 0`，Draggable 用 `x/y` 与边界 clamp；
  键盘方向键 8px、Shift 20px，Delete/Backspace 删除，焦点交给下一枚。
- 贴纸初始落点放在纸面边缘，不要压住正文与签名。

**导出成图**（`src/utils/exportPaper.js`，手写 canvas 合成，不用 html2canvas）：

- 3× 尺寸，纸外留 72px 透明边距；正文自动缩排直到能装下（字号循环递减到 11.5px）；
- 签名位置 `min(max(bodyBottom+22, height-92), height-58)`，防止跑出纸外；
- 纸张颜色跟随「奶油纸 / 雾蓝纸」主题；底图是一层浅亚麻桌面（渐变 + 经纬线 + 细框），
  这样发出去的是一张 flat-lay，而不是一张透明白图；
- 贴纸用序列化 SVG + 注入 CSS 的方式画到 canvas 上，只画落在纸面范围的；
- 文件名 `maison-des-mots-YYYYMMDD-HHmm.png`；导出中按钮 `aria-busy`，状态栏播报。

## 7. 动效规则

- **只保留有意义的动效**：换页入场、笔写作、书签抽出、示例卡 hover、贴纸落纸、便签展开。
  不要持续浮尘、无限旋转的圆环、全站指针视差。
- 时长：入场 350–700ms，总时长控制在 1.2–2.2s；hover 120–180ms；按钮/贴纸反馈 160–240ms。
- 入场必须「挨个出现」：每个元素 40–120ms stagger，先页眉/标题，再正文/卡片，最后贴纸与边角小物；
  不能整体一起淡入，也不能把入场动画删掉后什么都不补。
- 换页与入场不要分成两个阶段：换页当帧设新页、下一帧立即开始入场（见第 5 节）。
- 首屏视差只留给拼贴，`pointermove` 驱动 `gsap.quickTo`，入场完成后再启用，`pointerleave / pointercancel / window blur` 复位。
- 所有 GSAP 时间线用 `useGSAP` 的 context 管理；切换页面或依赖变化时 `revert/kill`，避免隐藏页面继续跑时间线。
- `prefers-reduced-motion: reduce`：不逐字、不视差、不做入场，直接显示终态，只保留很短的淡入与状态反馈。

## 8. 交互与可访问性清单

- [ ] 菜单：`aria-current="page"`、中文主标签、键盘可操作、焦点可见；≤1080 底部 tab 不遮挡内容与安全区
- [ ] 换页：hash 可前进后退；换页后焦点回到 `main`；无残留遮罩/克隆节点
- [ ] 空主题/空正文：字段附近 `role="alert"` 错误 + 焦点回到输入框，不用 toast
- [ ] 生成/书写中：按钮 `aria-busy`、状态 `role="status"`；「跳过书写」始终可用，切页/编辑时取消时间线
- [ ] 双模式：`aria-pressed`，模式切换后立即展示对应输入；自己写**不追加**任何模板句
- [ ] 复制：clipboard + 降级 textarea，状态播报；复制成功可有一枚短暂墨章
- [ ] 贴纸：`aria-pressed`、方向键/Delete、选中后焦点不丢；贴纸簿展开时 `aria-expanded` + 便签内可键盘操作
- [ ] 便签拖拽：只在页眉 `pointerdown` 生效，点按钮不触发拖拽；触屏 `touch-action: none` 只加在便签/贴纸上
- [ ] 导出：`aria-busy` + 成功/失败播报；导出图片包含当前纸色与贴纸
- [ ] 8 枚上限有提示；清空/取下后焦点留在触发按钮上（不要用 `disabled` 移除焦点）
- [ ] 跳至主要内容链接、`main` 的 `tabIndex=-1`、44px 触控目标、`:focus-visible`

## 9. 响应式断点（最容易做坏的地方）

| 断点 | 变化 |
| --- | --- |
| > 1280 | 封面两栏；吊牌栏竖排在左；书签竖排在右 |
| ≤ 1280 | 封面单栏；吊牌栏顶部横排；创作台两栏收窄 |
| ≤ 1080 | 书签改成底部 tab 条；`inner-frame` 预留 78px |
| 高度 ≤ 920 | 首屏 title/proof/间距收紧；创作台标题压成一行 |
| 高度 ≤ 800 | 创作台表单标题隐藏、textarea 收缩；灵感页隐藏卡片摘要 |
| 高度 ≤ 760 | 表单内滚、隐藏 local-note；灵感页隐藏 collection 摘要 |
| ≤ 760 | 手机：book-shell 贴边；创作台分「01 写作 / 02 纸页」两页签；贴纸簿便签浮层 |
| ≤ 480 | 隐藏版次小字；封面吊牌栏隐藏；灵感页卡片改横向条 |

坑位：

1. **整页滚动**：任何 `body`/`.app` 上的 `padding-bottom`（给底部 tab 留位的旧写法）都会制造 84px 滚动；改成 `inner-frame` 内边距。验收标准是 `document.scrollHeight === innerHeight`。
2. **换页后整页先出现再入场**：页面挂载时就 pause 在起点，靠 rAF 放开，见第 5 节。
3. **矮屏裁切**：一屏布局里 `min-height:0` 要一路传下去（shell → inner-frame → page-stage → page → flex 子项），少一层就撑破。
4. **花体压中文**：不要绝对定位让 Pinyon 横穿标题；放标题下方右对齐。
5. **绝对路径素材**：`public/assets/*.webp` 在 GitHub Pages 子路径会 404；用 `src/assets` + `import`。
6. **`touch-action: none` 死区**：只在选中贴纸/便签上使用。
7. **中文断行**：不要硬加 `<br>`，改字号/宽度。
8. **ResizeObserver 抖动**：回调内用 rAF 节流；页面隐藏（`clientWidth === 0`）时跳过。
9. **书签在矮屏被裁**：按视口高度缩放/收高度，不要写死。
10. **自己写被追加**：自定义模式只渲染 `customBody`，不要走模板补全；换行保留。
11. **字体子集**：接住用户输入的字体必须按常用字表子集，否则一输入生僻字就回退，观感割裂。
12. **导出与屏幕不一致**：字体、纸色、贴纸必须共用同一份配置表；导出前 `document.fonts.load`。

## 10. 交给 AI 的一句话提示词

把本文件作为系统约束，然后说：

> 用 `maison-des-mots-art-direction` 的方向，做一个「\[你的产品\]」的四页纸册应用。
> 保留：奶白 `#fbf9f4` + 银蓝 + 墨蓝的色板，Cormorant Garamond / Pinyon Script / Noto Serif SC / Special Elite / LXGW WenKai 五种字，
> 模切白边 + 1px 墨蓝线的纸感，右侧三层纸签菜单，**一屏一页不滚动**，换页用逐项入场，
> 首屏叠纸拼贴，创作台做一张浅色亚麻桌 + 可展开的贴纸簿，正文用毛笔逐组显影。
> 功能是 \[一句话功能\]。先给视觉方案，再落 React + Vite + GSAP；所有素材与字体必须本地化。

更省事的做法：直接让它读本仓库的 `src/styles.css`、`src/components/HeroCollage.jsx`、
`src/components/Studio.jsx`、`src/components/MenuRibbons.jsx`、`src/utils/exportPaper.js`，
按同样的 token 换内容，而不是重新发明一套纸。

## 11. 验收

- [ ] 1440 / 1280 / 1080 / 768 / 390 / 320 无整页滚动、无横向滚动、无被裁正文
- [ ] 1440×900 / 1280×720 / 390×844 三个基准尺寸下，四页都在一屏内
- [ ] 四个页面与书签全部可达；返回/前进可用；换页后焦点回到 main
- [ ] 从任意页点菜单回封面，能看到完整入场动画（不是静态出现）
- [ ] 首屏标题 2 行内可读；花体不压字；01/02/03 那行不与横线相碰
- [ ] 自己写：标题可空、正文原样、换行保留、无任何追加；「编辑这段」能带回当前纸面
- [ ] 笔写作可跳过、切页会取消、速度三档可用、reduced-motion 直接显示全文
- [ ] 贴纸簿：展开/收起、便签可拖、点选落纸、拖拽落纸、8 枚上限
- [ ] 导出：奶油/雾蓝两种纸色都生效；3× 尺寸；字幕/签名不出纸；贴纸一起带走
- [ ] 生成 → 书写 → 贴纸 → 复制 → 导出 → 清空全链路可用；键盘可完成
- [ ] reduced-motion / forced-colors / 200% zoom 不崩
- [ ] 部署在子路径时素材全部加载；字体已本地化，断网不出外部字体请求
