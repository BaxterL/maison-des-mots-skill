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
  --ink-soft: #4f5c7c;     /* 次级正文 */
  --fog: #66718f;          /* 注释、编号 */
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
- 中文：`Noto Serif SC` 400–700
- 法文花体：`Pinyon Script`（一句即可，别超过 2 处）
- 档案/编号/注脚：`Special Elite`（替代 Courier 的更旧打字机感）

**纸的物理感**：所有卡片共用一套阴影，不要发明第二种。

```css
--paper-shadow:
  0 0 0 3px rgba(255,255,255,.88),   /* 模切白边 */
  0 0 0 4px rgba(30,39,72,.12),      /* 细墨蓝外线 */
  0 16px 30px rgba(30,39,72,.14);    /* 短距离柔阴影 */
```

**比例**：标题字重 500、行高 1.24–1.45、字距 -0.055em（中文大字）到 +0.06em（正文）。
正文 13–15px / 行高 2；微型注脚不低于 9px，且在浅底上保持 `--ink-soft`。

## 3. 页面骨架

```
book-shell（最大 1680，外边距 22）
└─ inner-frame（内缩 15，1px hairline 边框）
   ├─ 四角 SVG 花角（aria-hidden，绝对定位）
   ├─ masthead（3 列：品牌 / 导航 / 版次）
   ├─ main
   │  ├─ hero（左侧 92px 吊牌栏 + 文案 + 430px 信笺拼贴）
   │  ├─ hero-ledger（一行法文手写体 + 两端打字机小字）
   │  ├─ studio（左表单 / 右画布）
   │  ├─ inspiration（左侧标题 + 右侧档案列表）
   │  └─ about（居中一段话 + 素材来源）
   └─ footer（3 段小字，中间品牌）
```

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
4. 邮票、玫瑰、山茶花吊牌、挂着的香水瓶、Souvenir 菱形卡、星芒

信笺内部：`LETTRE N° 001 / PARIS · 2026` → 法文斜体小句 → 中文 33px 两行 → Pinyon 手写体 →
11px 正文 → 拱廊版画（600×392，`mix-blend-mode: multiply`，opacity .82）→ 页脚。

**主标题**：两行中文 `52–80px`（`clamp(50px, 4.9vw, 78px)`），每行放进 `overflow:hidden` 的 mask，
入场时 `yPercent: 115 → 0`。花体 `Les mots doux` 放在标题之后的独立一行、右对齐、rotate -6°，
**不要**压在主标题上——中文被花体穿过会直接不可读。

## 5. 创作台配方

- 左表单：奶白叠纸 + 7px 藕粉纸胶带，字段用 `01 / 02` 档案编号，textarea 带 30px 横线背景。
- 右画布：`min-height 610`，四角 L 形边框，中央信笺 `width 64%`，`min-height 470`。
- 生成时：信笺先 `opacity 0 / y 24 / rotation -4` 退场，换文案后再从上方落下；
  正文用 `{value: 0 → length}` 计数字典逐字写入，`onComplete` 解锁按钮。
- 贴纸：6 类手绘线稿（统一 `1.1` 描边、墨蓝+银蓝两色），按钮 92×110，`filter: drop-shadow(3px 6px 4px …)`；
  选中态只加虚线边 + 极少量藕粉，不加粗、不换尺寸（避免 Draggable 坐标错位）。
- 画布上的贴纸必须 `position:absolute; inset:0 auto auto 0`，Draggable 用 `x/y` 与边界 clamp；
  键盘方向键 8px、Shift 20px，Delete/Backspace 删除，焦点交给下一枚或贴纸架。

## 6. 动效规则

- 入场只做一次，顺序：吊牌栏 → 版次/编号 → 标题逐行 → 花体 → 描述/按钮 → 表盘 → 雾蓝纸 → 信笺 → 贴纸/吊牌 → Souvenir。
- 时长 .45–1.0s，`power3.out`，错位 stagger .07–.11；不要用弹性过猛的 ease。
- 首屏视差：`pointermove` 驱动 `gsap.quickTo`，每层深度 2.5–7.5px，入场完成后再启用；`pointerleave / pointercancel / window blur` 全部复位。
- 禁止无限循环动画；`prefers-reduced-motion: reduce` 下取消入场、视差、循环与逐字，直接呈现终态。
- 动效只用 transform / opacity；不要在一次入场里同时动 `top/left` 和 `x/y`。

## 7. 交互与可访问性清单

- [ ] 空主题：`role="alert"` 错误 + 焦点回到 textarea
- [ ] 生成中：按钮 `aria-busy`、状态 `role="status"`，重复点击不叠加时间线
- [ ] 复制：clipboard + 降级 textarea 方案，状态播报成功/失败
- [ ] 贴纸：`aria-pressed`、序号读屏名、方向键/Delete、选中后焦点不丢
- [ ] 8 枚上限有提示；清空/取下后焦点留在触发按钮上（**不要用 `disabled` 移除焦点**）
- [ ] `prefers-reduced-motion` 下无动画锁死
- [ ] 法文片段标 `lang="fr"`；装饰性 SVG `aria-hidden`
- [ ] 触屏：画布内未选中的贴纸 `touch-action: manipulation`，选中后才 `none`，避免页面滑不动
- [ ] 跳至主要内容链接、`main` 的 `tabIndex=-1`、44px 触控目标、`:focus-visible`

## 8. 响应式断点（这是最容易做坏的地方）

| 断点 | 变化 |
| --- | --- |
| > 1280 | 首屏两栏；吊牌栏竖排在左 |
| ≤ 1280 | 首屏单栏：文案块居中 max 680，拼贴宽 650；吊牌栏变成顶部横排纸签 |
| ≤ 1080 | 创作台单栏；表单与画布上下排 |
| ≤ 640 | 手机：书壳内缩、字号下调、贴纸架横向滚动 |
| ≤ 480 | 隐藏版次第二行小字 |
| ≤ 370 | 语气选项改单列 |

坑位：

1. **花体压中文**：不要用绝对定位让 Pinyon 横穿标题；放标题下方右对齐。
2. **左侧吊牌栏与文案重叠**：竖排栏只在 >1280 存在；≤1280 必须回到文档流。
3. **窄桌面（1081–1280）两栏挤压**：视觉列最小 430px 仍可能让标题折行，直接改单栏最稳。
4. **绝对路径素材**：`public/assets/*.webp` 在 GitHub Pages 子路径会 404；改为 `src/assets` + `import`，由 Vite 处理 base。
5. **`touch-action: none` 死区**：只在选中贴纸上使用。
6. **中文断行**：不要硬加 `<br>`，改字号/宽度；小屏标题 `clamp(39px, 11vw, 56px)`。
7. **ResizeObserver 抖动**：回调内用 `requestAnimationFrame` 节流；打字动画期间不要反复推贴纸。

## 9. 交给 AI 的一句话提示词

把本文件作为系统约束，然后说：

> 用 `maison-des-mots-art-direction` 的方向，做一个「\[你的产品\]」的单页应用。
> 保留：奶白 `#fbf9f4` + 银蓝 + 墨蓝的色板，Cormorant Garamond / Pinyon Script / Noto Serif SC / Special Elite 四种字，
> 模切白边 + 1px 墨蓝线的纸感，左侧吊牌栏，右侧叠纸信笺，入场时间线与 reduced-motion。
> 功能是 \[一句话功能\]。先给视觉方案，再落 React + Vite + GSAP；所有素材必须本地化并标注授权。

更省事的做法：直接让它读本仓库的 `src/styles.css`、`src/components/HeroCollage.jsx`、
`src/components/Studio.jsx`，按同样的 token 换内容，而不是重新发明一套纸。

## 10. 验收

- [ ] 1440 / 1280 / 1080 / 768 / 390 / 320 无横向滚动、无遮挡、无被裁正文
- [ ] 首屏标题 2 行内可读；花体不压字；吊牌栏不盖文案
- [ ] 生成 → 打字 → 贴纸 → 复制 → 清空全链路可用；键盘可完成
- [ ] reduced-motion / forced-colors / 200% zoom 不崩
- [ ] 部署在子路径时素材全部加载（检查 `dist/assets/*.webp` 与 `import.meta.env.BASE_URL`）
- [ ] 素材来源与字体授权写进 README

---

素材：Trianon（CC0）的纸样与山茶花、The Met 的公有领域拱廊版画；字体均为 Google Fonts。
仅参考视觉语言，不复制 Trianon 的业务代码、签名或水印。
