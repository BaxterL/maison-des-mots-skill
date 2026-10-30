---
name: maison-des-mots-art-direction
description: 设计并实现「法式洛可可纸雕 / vintage scrapbook」网页 UI。适用于奶白与银蓝配色、模切白边、叠纸信笺、吊牌、手绘线稿贴纸、打字机档案标签、法文花体、GSAP 入场与拼贴交互；也适用于把任何工具类页面改造成一页可翻阅的纸上作品。
license: MIT
---

# Maison des Mots · 法式纸雕网页 Art Direction

一套已经落地验证过的美术方向与实现配方。目标不是「加点复古元素」，而是让整页像一件用纸、线和阴影做出来的实物：**克制的颜色、真实纸感、清晰的层级，以及可以触摸的交互。**

参考实现：`https://github.com/BaxterL/maison-des-mots`（React 19 + Vite + GSAP）。
本文件可整份交给 AI 编码助手，按第 9 节改一句话即可开工。

---

## 1. 先定方向

一句话：**把一间凡尔赛宫的纸艺工作室，装进一个安静的浏览器窗口。**

- 是「纸雕 + 档案检索 + 手账拼贴」，不是泛法式复古，也不是 AI 工作台。
- 高级感来自留白、1px 线、错位叠纸和一致的模切白边，不来自花纹、金边、深色底。
- 每屏只允许一个视觉主角；装饰必须能被忽略，不能被误认为控件。
- 中文负责叙事，法文负责气质，英文衬线负责标题，打字机字体负责编号与档案注脚。

**不要**：暖金、大面积深色、密集波点/网格/噪点、粗野硬阴影、持续旋转的装饰、满屏手写体。

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
  --lotus: #e8c9d0;        /* 藕粉，只用一点点：选中、星芒、一枚标签 */
  --line: rgba(30, 39, 72, .42);
  --hairline: rgba(30, 39, 72, .16);
}
```

**字体**（Google Fonts，全部 SIL OFL）：

- 标题/英文衬线：`Cormorant Garamond` 400–600 + italic
- 中文标题：`Noto Serif SC` 400–700
- 信笺正文 / 灵感卡片：`LXGW WenKai`（霞鹜文楷）500 / 700
- 法文花体：`Pinyon Script`（一句即可，别超过 2 处）
- 档案/编号/注脚：`Special Elite`（替代 Courier 的更旧打字机感）

> **分层原则**：WenKai 只给“信/手写”的内容（信笺正文、示例纸、宣言），
> 表单、计数、状态、操作标签继续用清晰的宋体；不要让整个界面都变软。

> 中国大陆网络下 Google Fonts 经常不可用。上线前把字体本地化：拉丁字体直接用 woff2，
> 中文用 `fontTools.subset` 按页面实际用字做子集（本项目 353 个汉字，变量字体约 206KB），
> 页面就完全没有外部字体请求。

**纸的物理感**：所有卡片共用一套阴影，不要发明第二种。

```css
--paper-shadow:
  0 0 0 3px rgba(255,255,255,.88),   /* 模切白边 */
  0 0 0 4px rgba(30,39,72,.12),      /* 细墨蓝外线 */
  0 16px 30px rgba(30,39,72,.14);    /* 短距离柔阴影 */
```

**比例**：标题字重 500、行高 1.24–1.45、字距 -0.055em（中文大字）到 +0.06em（正文）。
正文 14–16px / 行高 1.9–2；表单与状态文字不低于 11–13px；微型注脚不低于 9px，且在浅底上保持 `--ink-soft`。

## 3. 页面骨架

```
app
├─ page-texture（全屏 grain，2%）
├─ book-shell（最大 1680，外边距 22）
│  └─ inner-frame（内缩 15，1px hairline 边框）
│     ├─ 四角 SVG 花角（aria-hidden，绝对定位）
│     ├─ masthead（品牌 / 当前章节 / 版次）
│     ├─ menu-ribbons（右缘 4 枚书签；≤1080 固定底部 tab）
│     ├─ main.page-stage
│     │  └─ .page-view[hidden] × 4（懒挂载，保留状态）
│     │     ├─ 01 couverture：hero + hero-ledger
│     │     ├─ 02 atelier：studio（表单 + 纸 + 贴纸架）
│     │     ├─ 03 inspiration：3 张示例纸 + 贴纸样本册
│     │     └─ 04 maison：居中的“纸房间”宣言
│     └─ footer（3 段小字，中间品牌）
└─ PageTurn（fixed，全屏纸页翻转，z-index 200）
```

- 路由用 hash（`#/atelier`）；切页时先设新页、滚动到顶部，再让纸页翻转，转完触发新页入场。
- `.book-shell` 用 `outline` 做内侧白边、`box-shadow` 做漂浮；`overflow: clip` 防止装饰溢出。
- 背景用 3 层低对比 radial-gradient 模拟云石晕染（银蓝 + 一点藕粉），不要用图片。
- 纸张颗粒用一个全屏 `<svg>` + `feTurbulence`，透明度压到 2% 左右，`mix-blend-mode: multiply`。

## 4. 首屏配方

**吊牌栏**（桌面）：左侧 92px 宽、绝对定位，一条银蓝渐变细绳从上挂到底；
5 枚模切纸签（香水瓶、缎带、玫瑰、钥匙、月亮），宽度 64 / 高度 82，纸签带顶部圆孔和 1px 内框，
各自 rotate(-4°…4°)。整条 `aria-hidden="true"`、`pointer-events: none`。
≤1280px 时改成页面顶部横排纸签（高度 88，绳子变成水平线）。

**拼贴**：从后到前是

1. 档案表盘（540px 圆，虚线环 + 48 根刻度 + `textPath` 环形法文，opacity .55）
2. 雾蓝纸（69% × 78%，斜切 clip-path，rotate 5°）
3. 奶油信笺（`min(68%, 430px)`，min-height 550，内缩 9px 的 hairline，rotate -1.5°）
4. 邮票、山茶花吊牌、挂着的香水瓶、Souvenir 菱形卡

> 桌面首屏大约 10 个装饰件；超过这个数就会满。删掉玫瑰与星芒两件之后，标题和信笺才重新有呼吸。
> 装饰只放在拼贴四周，不要压住正文与主按钮。

信笺内部：`LETTRE N° 001 / PARIS · 2026` → 法文斜体小句 → 中文 33px 两行 → Pinyon 手写体 →
11px 正文 → 拱廊版画（600×392，`mix-blend-mode: multiply`，opacity .82）→ 页脚。

**主标题**：两行中文 `52–80px`（`clamp(50px, 4.9vw, 78px)`），每行放进 `overflow:hidden` 的 mask，
入场时 `yPercent: 115 → 0`。花体 `Les mots doux` 放在标题之后的独立一行、右对齐、rotate -6°，
**不要**压在主标题上——中文被花体穿过会直接不可读。

## 5. 书签菜单与翻页

**不要做传统横条导航。** 四个目的地对应右缘四枚竖排书签：

- 位置：内框右侧 13px、顶部 152px；宽 42、高 162，底部斜切（`clip-path`），顶部一枚圆孔。
- 文案：中文主标签 12px，编号 9px、法文 8px 退后；active 向左抽出 15px 并换成墨色，hover 抽出 9px。
- 屏幕高度 < 940px 时隐藏法文，书签高 118，保证四枚都在视口内；≤1080 改成固定底部纸质 tab 条（高 50，安全区之上，页面底部留 78px）。
- `aria-current="page"`、`aria-label` 带编号与中法文；hover 只是位移，不叠额外动效。

**翻页转场**（比“开门”更贴合纸的语义）：

- 全屏纸页，`perspective: 2200px`；前进时 `transform-origin: left center`，后退时右侧装订边。
- `rotateY` 0 → ±178°，650–820ms，`power2.inOut`；背面是 `rotateY(180deg)` 的下一章，纸背略暗。
- 随角度变化的 `.turn-shade` 只做一层阴影，**不要把全屏压灰**；装订边留 1–3px 纸厚，边缘用 inset shadow。
- 转场结束后新页才开始 mask/stagger 入场；`prefers-reduced-motion` 直接切页，不渲染遮罩。
- 用 hash 路由 + 懒挂载页面；`overflow` 的页面不要在创作台里再套一层滚动。

## 6. 创作台与笔写作

- 左表单：奶白叠纸 + 7px 藕粉纸胶带，字段用 `01 / 02` 档案编号，textarea 带 30px 横线背景。
- 右画布：`min-height 570`，四角 L 形边框，中央信笺 `width 64%`，`min-height 440`。
- **双模式**：
  - `用灵感起稿`：主题 + 语气 → 模板补全成一封信；
  - `写自己的`：标题可选、正文必填 → 原样上纸，**绝不追加任何模板句子**。
  - 「编辑这段」把当前纸面文字带回编辑区；空正文在字段附近报错，不用 toast。
- **笔写作**（替代打字机）：
  - 正文按 2–3 个 token 分组，标点/换行作为停顿；组内 opacity + 轻微 blur 显影，组间约 0.17s。
  - 羽毛笔跟随每组最后一个非空 token，笔尖落在字的基线末端，尺寸小于字高；标题不做逐字，先落标题再写正文。
  - 写完笔淡出；提供「跳过书写」，切页/编辑时立即取消时间线；`prefers-reduced-motion` 直接显示全文。
- 贴纸：6 类手绘线稿（统一 `1.1` 描边、墨蓝+银蓝两色），按钮 92×110，`filter: drop-shadow(3px 6px 4px …)`；
  选中态只加虚线边 + 极少量藕粉，不加粗、不换尺寸（避免 Draggable 坐标错位）。
- 画布上的贴纸必须 `position:absolute; inset:0 auto auto 0`，Draggable 用 `x/y` 与边界 clamp；
  键盘方向键 8px、Shift 20px，Delete/Backspace 删除，焦点交给下一枚或贴纸架。
- 贴纸初始落点放在纸面边缘，不要压住正文与签名。

## 7. 动效规则

- **只保留有意义的动效**：页面翻页、笔写作、书签抽出、示例卡 hover、首屏叠纸入场。
  不要持续浮尘、无限旋转的圆环、全站指针视差——装饰应该响应用户，而不是自己表演。
- 时长：页面转场 650–820ms；入场不超过 450ms；hover 120–180ms；按钮/贴纸反馈 160–240ms。
- 页面转场与内容入场不要同时开始：翻页结束再让新页 mask/stagger 出现；一个时刻只让一个主物件动。
- 首屏视差只留给拼贴，`pointermove` 驱动 `gsap.quickTo`，入场完成后再启用，`pointerleave / pointercancel / window blur` 复位。
- 所有 GSAP 时间线用 `useGSAP` 的 context 管理；切换页面或依赖变化时 `revert/kill`，避免隐藏页面继续跑时间线。
- `prefers-reduced-motion: reduce`：不翻页、不逐字、不视差、不循环；直接显示终态，只保留很短的淡入与状态反馈。

## 8. 交互与可访问性清单

- [ ] 菜单：`aria-current="page"`、中文主标签、键盘可操作、焦点可见；≤1080 底部 tab 不遮挡内容与安全区
- [ ] 页面：`PageTurn` 标 `aria-hidden`；转场结束焦点回到 `main`；hash 可前进后退
- [ ] 空主题/空正文：字段附近 `role="alert"` 错误 + 焦点回到输入框，不用 toast
- [ ] 生成/书写中：按钮 `aria-busy`、状态 `role="status"`；**「跳过书写」**始终可用，切页/编辑时取消时间线
- [ ] 双模式：`aria-pressed`，模式切换后立即展示对应输入；自己写**不追加**任何模板句
- [ ] 复制：clipboard + 降级 textarea 方案，状态播报成功/失败；复制成功可有一枚短暂的墨章反馈
- [ ] 贴纸：`aria-pressed`、序号读屏名、方向键/Delete、选中后焦点不丢
- [ ] 8 枚上限有提示；清空/取下后焦点留在触发按钮上（**不要用 `disabled` 移除焦点**）
- [ ] 触屏：画布内未选中的贴纸 `touch-action: manipulation`，选中后才 `none`，避免页面滑不动
- [ ] 跳至主要内容链接、`main` 的 `tabIndex=-1`、44px 触控目标、`:focus-visible`

## 9. 响应式断点（这是最容易做坏的地方）

| 断点 | 变化 |
| --- | --- |
| > 1280 | 封面两栏；吊牌栏竖排在左；书签竖排在右 |
| ≤ 1280 | 封面单栏：文案块居中 max 680，拼贴宽 650；吊牌栏变成顶部横排纸签 |
| ≤ 1080 | 创作台单栏；表单与画布上下排；书签改成固定底部 tab 条 |
| 高度 ≤ 940 | 桌面书签隐藏法文、高度收到 118，确保四枚都在视口内 |
| ≤ 640 | 手机：书壳内缩、字号下调、贴纸架横向滚动、转场缩短到 620ms |
| ≤ 480 | 隐藏版次第二行小字；封面的吊牌栏改为隐藏，让主按钮更早出现 |
| ≤ 370 | 语气选项改单列 |

坑位：

1. **花体压中文**：不要用绝对定位让 Pinyon 横穿标题；放标题下方右对齐。
2. **左侧吊牌栏与文案重叠**：竖排栏只在 >1280 存在；≤1280 必须回到文档流；≤480 直接隐藏。
3. **窄桌面（1081–1280）两栏挤压**：视觉列最小 430px 仍可能让标题折行，直接改单栏最稳。
4. **绝对路径素材**：`public/assets/*.webp` 在 GitHub Pages 子路径会 404；改为 `src/assets` + `import`，由 Vite 处理 base。
5. **`touch-action: none` 死区**：只在选中贴纸上使用。
6. **中文断行**：不要硬加 `<br>`，改字号/宽度；小屏标题 `clamp(39px, 11vw, 56px)`。
7. **ResizeObserver 抖动**：回调内用 `requestAnimationFrame` 节流；页面隐藏（`clientWidth === 0`）时直接跳过，否则贴纸会被压到 0,0。
8. **书签在矮屏被裁**：按视口高度收高度/隐藏法文，不要把书签写死 162px。
9. **底部 tab 遮内容**：给 `.app` 留 78px 底部空间，并处理 `env(safe-area-inset-bottom)`。
10. **自己写被追加**：自定义模式只渲染 `customBody`，不要走模板补全函数；换行用 `<br>` 或 token 换行保留。

## 10. 交给 AI 的一句话提示词

把本文件作为系统约束，然后说：

> 用 `maison-des-mots-art-direction` 的方向，做一个「\[你的产品\]」的四页纸册应用。
> 保留：奶白 `#fbf9f4` + 银蓝 + 墨蓝的色板，Cormorant Garamond / Pinyon Script / Noto Serif SC / Special Elite / LXGW WenKai 五种字，
> 模切白边 + 1px 墨蓝线的纸感，右侧书签菜单 + 纸页翻转，首屏叠纸拼贴，正文用羽毛笔逐组显影。
> 功能是 \[一句话功能\]。先给视觉方案，再落 React + Vite + GSAP；所有素材与字体必须本地化。

更省事的做法：直接让它读本仓库的 `src/styles.css`、`src/components/HeroCollage.jsx`、
`src/components/Studio.jsx`、`src/components/PageTurn.jsx`，按同样的 token 换内容，而不是重新发明一套纸。

## 11. 验收

- [ ] 1440 / 1280 / 1080 / 768 / 390 / 320 无横向滚动、无遮挡、无被裁正文
- [ ] 四个页面与右缘书签全部可达；返回/前进可用；转场结束焦点回到 main
- [ ] 矮屏（800 高）书签不越出视口；≤1080 底部 tab 不遮内容与安全区
- [ ] 首屏标题 2 行内可读；花体不压字；吊牌栏不盖文案
- [ ] 自己写：标题可空、正文原样、换行保留、无任何追加；编辑这段能带回当前纸面
- [ ] 笔写作可跳过、切页会取消、reduced-motion 直接显示全文
- [ ] 生成 → 书写 → 贴纸 → 复制 → 清空全链路可用；键盘可完成
- [ ] reduced-motion / forced-colors / 200% zoom 不崩
- [ ] 部署在子路径时素材全部加载（检查 `dist/assets/*.webp` 与 `import.meta.env.BASE_URL`）
- [ ] 字体已本地化，断网/大陆网络下页面不出外部字体请求
