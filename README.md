# Maison des Mots Skill

**法式洛可可纸雕网页的美术方向，写成一份可以交给 AI 的 Skill。**

![Maison des Mots](images/hero.jpg)

这套视觉来自一个真实落地的项目：[Maison des Mots · 字里行间](https://github.com/BaxterL/maison-des-mots)
（[在线参观](https://baxterl.github.io/maison-des-mots/)）——一个开在浏览器里的法式纸雕文案室。

`SKILL.md` 把它的颜色、字体、纸的物理感、组件配方、动效、可访问性清单、响应式断点和踩坑记录，
整理成 AI 编码助手可以直接执行的约束。不是 moodboard，是一份能长出页面的施工图。

## 怎么用

**Claude Code / 任意支持 Skill 的 agent**：把 `SKILL.md` 放进 skills 目录，或直接让它读这份文件。

**只想快点开始**：把下面这句和 `SKILL.md` 一起丢给 AI。

> 用 `maison-des-mots-art-direction` 的方向，做一个「[你的产品]」的单页应用。
> 保留奶白 + 银蓝 + 墨蓝的色板，Cormorant Garamond / Pinyon Script / Noto Serif SC / Special Elite 四种字，
> 模切白边 + 1px 墨蓝线的纸感，左侧吊牌栏，右侧叠纸信笺，入场时间线与 reduced-motion。
> 功能是 [一句话功能]。先给视觉方案，再落 React + Vite + GSAP；所有素材必须本地化并标注授权。

## 里面有什么

- **设计 token**：完整色板、字体组合、纸的阴影公式。
- **页面骨架**：书壳、花角、报头、首屏拼贴、创作台、灵感档案、关于。
- **组件配方**：模切信笺、纸胶带、邮戳、Souvenir 菱形卡、吊牌、六枚线稿贴纸。
- **动效规则**：入场时间线顺序、视差、打字机、贴纸，以及 reduced-motion。
- **交互与可访问性**：焦点、触控、键盘、8 枚上限、状态播报。
- **响应式断点**：从 1920 到 320 的完整策略，以及最容易做坏的 5 个坑。
- **验收清单**：交付前逐条对照。

## 预览

![创作台](images/studio.jpg)

![拼贴细节](images/detail.jpg)

## 素材与授权

- 纸样与山茶花来自 [Trianon](https://github.com/qimingjiu/Trianon)，作者声明为 CC0。
- 拱廊版画为 The Met 公有领域馆藏（Francesco Guardi）。
- 字体来自 Google Fonts（SIL OFL）：Cormorant Garamond、Pinyon Script、Noto Serif SC、Special Elite。

`SKILL.md` 与示例图以 MIT 发布；第三方素材遵循其原始授权。
