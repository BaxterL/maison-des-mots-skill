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

> 用 `maison-des-mots-art-direction` 的方向，做一个「[你的产品]」的四页纸册应用。
> 保留奶白 + 银蓝 + 墨蓝的色板，Cormorant Garamond / Pinyon Script / Noto Serif SC / Special Elite / LXGW WenKai 五种字，
> 模切白边 + 1px 墨蓝线的纸感，右侧书签菜单 + 纸页翻转，首屏叠纸拼贴，正文用羽毛笔逐组显影。
> 功能是 [一句话功能]。先给视觉方案，再落 React + Vite + GSAP；所有素材与字体必须本地化。

## 里面有什么

- **设计 token**：完整色板、字体组合、纸的阴影公式。
- **页面骨架**：书壳、花角、报头、四页内容、右缘书签菜单、懒挂载页面。
- **组件配方**：模切信笺、纸胶带、邮戳、Souvenir 菱形卡、吊牌、六枚线稿贴纸。
- **转场与笔写作**：650–820ms 的纸页翻转、分组显影的羽毛笔、跳过与 reduced-motion。
- **文案逻辑**：「用灵感起稿 / 写自己的」双模式，自己写绝不追加模板句。
- **动效规则**：入场顺序、书签抽出、hover 反馈，以及什么时候该删动效。
- **交互与可访问性**：焦点、触控、键盘、8 枚上限、状态播报。
- **响应式断点**：从 1920 到 320、矮屏书签、底部纸质 tab。
- **验收清单**：交付前逐条对照。

## 预览

![创作台](images/studio.jpg)

![笔写作](images/pen.jpg)

![拼贴细节](images/detail.jpg)
