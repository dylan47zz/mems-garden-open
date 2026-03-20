---
categories: [tools, setups]
tags: [obsidian, pkm, workflow]
created: 2026-03-19
status: evergreen
related: []
publish: true
---

# 我的 Obsidian 使用指南

> 基于 [kepano (Steph Ango)](https://stephango.com/vault) 方法论，结合个人使用场景定制。
> 核心原则：**拥抱混沌与懒惰，创造涌现结构**。

---
Xd20250430zz@@
## 方法论

### 核心哲学

- **File over App** — 笔记是我的，格式是标准 Markdown，不绑定任何工具，文件可以永久存活
- **Bottom-up** — 不预设分类体系，让结构从内容中自然涌现，避免过度设计
- **Speed & Laziness** — 系统的目的是**降低摩擦**，而不是增加仪式感。能不分类就不分类

### 两种笔记的区分

| 存放位置 | 含义 | 例子 |
|----------|------|------|
| `笔记/` | 我亲自写的、与我直接相关的 | 日记碎片、常青笔记、月回顾 |
| `参考资料/` | 外部世界存在的事物 | 书、工具、论文、人物 |
| `剪藏/` | 他人写的内容 | 文章、引用、摘录 |

### Fractal Journaling（碎片→沉淀）

```
当天随时              每几天               每月               每年
──────────────       ──────────────       ──────────────     ──────────────
思维碎片笔记      →   日记碎片汇总      →  月度回顾       →  年度回顾
YYYY-MM-DD HHmm      YYYY-MM-DD 日记      YYYY-MM 月回顾     YYYY 年回顾
```

1. 随时用 `Unique Note` 快捷键，创建带时间戳的碎片笔记，存入 `笔记/`
2. 每隔几天回顾碎片，整理成日记汇总
3. 每月底写月回顾，引用重要日记
4. 每年底用 [40个自我追问](https://stephango.com/40-questions) 做年度回顾

### 随机漫游

每隔几个月，用 `Random Note` 快捷键随机穿行 Vault，结合 Local Graph（浅层）发现旧连接，做笔记维护。**不要把这件事交给 AI**，自己做才能理解自己的思维模式。

---

## Vault 结构（两库统一）

```
vault/
├── 收件箱/   ← 快速捕获，定期清空
├── 任务/     ← 所有 TaskNote（one task one note）
│   └── 视图/ ← Bases 视图文件
├── 笔记/     ← 你写的一切：常青笔记、日记碎片、文章、项目
├── 剪藏/     ← 外部内容：文章、书摘、工具、人物
├── 分类/     ← 各类目 Dataview 索引页（不存笔记）
├── 日记/     ← 每日入口 / 日期锚点
├── 模板/     ← 所有笔记模板
└── 附件/     ← 图片、PDF、附件
```

**设置**：Settings → Files and links → Default location for new notes → `收件箱`

---

## 个人规则（Style Guide）

| 规则 | 说明 |
|------|------|
| 单一 Vault | 所有内容在一个 Vault，不拆分 |
| 避免深层文件夹 | 最多 2 层，靠 `categories` 属性组织 |
| 标准 Markdown | 不用非标准语法，保证可移植性 |
| 标签/分类复数 | `tools` 不是 `tool`，`papers` 不是 `paper` |
| 大量使用内链 | 第一次提到某事物就 `[[链接]]`，未创建的页面也可以链 |
| 日期格式统一 | 全用 `YYYY-MM-DD` |
| 评分 1-7 分制 | 见下方评分系统 |
| 属性名简短 | `author` 不是 `note-author` |

---

## 分类体系（Categories）

用 YAML `categories` 属性组织笔记，而非文件夹。一篇笔记可以属于多个分类。

### 🧠 知识学习
- `ai` — AI 技术、模型、论文、思考
- `recommender-systems` — 推荐算法、召回、排序、业界实践
- `engineering` — 工程实践、架构、系统设计
- `papers` — 学术论文精读

### 🛠 效率工具
- `tools` — 软件工具、插件、配置方案
- `setups` — 工作流设置、dotfiles、环境配置
- `shortcuts` — 快捷键、命令、脚本备忘

### 💭 个人思考
- `essays` — 个人文章、观点
- `journal` — 日记、思维碎片汇总
- `evergreen` — 常青笔记，经过时间检验的观点
- `questions` — 待思考的问题

### 📚 外部资源
- `books` — 读过/在读的书
- `articles` — 外部文章
- `people` — 值得关注的人
- `projects` — 开源项目、产品

---

## 属性规范（Properties / Frontmatter）

```yaml
---
categories: [ai, papers]     # 分类，复数，列表类型
tags: [transformer, llm]     # 细粒度标签
created: 2026-03-19
author: [[作者名]]            # 人物链接
rating: 6                    # 1-7 分制
status: reading              # reading / done / abandoned / evergreen
related: [[相关笔记]]
---
```

**属性原则**：
- 属性名跨分类复用（如 `genre` 同时用于书和电影，可以跨类聚合）
- 模板可组合，如一篇笔记可以同时套用 `Person` 和 `Author` 模板
- 优先用 `list` 类型，而非 `text`，方便未来扩展

---

## 模板列表

| 模板 | 适用场景 |
|------|----------|
| `日记碎片` | 随时捕捉的思维碎片 |
| `常青笔记` | 经过提炼的原子观点 |
| `论文笔记` | 学术论文精读 |
| `书籍笔记` | 读书记录 |
| `工具笔记` | 效率工具/配置记录 |
| `AI笔记` | AI 技术学习笔记 |

**设置**：Settings → Templates → Template folder location → `模板`（当前使用核心 Templates 插件）

---

## 评分系统（1-7 分）

| 分值 | 评级 | 含义 |
|------|------|------|
| 7 | Perfect | 改变人生，必须体验，反复推荐 |
| 6 | Excellent | 非常值得，会再次回顾 |
| 5 | Good | 不错，有价值 |
| 4 | Passable | 勉强可以 |
| 3 | Bad | 不推荐 |
| 2 | Atrocious | 主动回避 |
| 1 | Evil | 有害 |

> 选 7 分而非 10 分：顶部需要足够的粒度区分好的体验，10 分过于冗余。

---

## 主题与外观

| 配置项 | 当前值 |
|--------|--------|
| **主题** | Baseline（已安装：Baseline, Kabadoni） |
| **配色方案** | Adwaita Light / Adwaita Dark |
| **界面字体** | Adwaita Sans |
| **等宽字体** | Adwaita Mono |
| **布局风格** | Minimal layout + Floating tabs |
| **标题样式** | 彩色标题，逐级递减（H1 2em → H6 1.25em） |

### CSS Snippets

| Snippet | 用途 |
|---------|------|
| `Dashboard-Brutalist` | Dashboard 页面的 Brutalist 风格布局 |
| `kabadoni-fixes` | Kabadoni 主题的浅色模式修复（表头、标题对比度等） |

### Style Settings 关键调整

通过 Style Settings 插件对 Baseline 主题做了大量自定义：
- 密度 1.25x，圆角 1x，动画速度 2x
- 自定义代码高亮配色（Adwaita 风格）
- 彩色标题（`colorful-headings-on`），加粗权重 800/700
- 文件头始终显示、左对齐、700 粗体
- 隐藏 Vault 切换器和标题栏文字
- 图片网格布局、深色模式 PDF 反色

---

## 插件配置

### 当前启用的第三方插件

| 插件 | 版本 | 用途 |
|------|------|------|
| **Dataview** | 0.5.68 | 按属性动态查询笔记，驱动分类索引页 |
| **BRAT** | 2.0.2 | 安装和测试 beta 版插件 |
| **Style Settings** | 1.0.9 | 调整主题、插件和 Snippet 的 CSS 变量 |
| **Colored Tags** | 6.1.2 | 为标签自动着色，嵌套标签混合父级颜色 |
| **Notebook Navigator** | 2.5.1 | 双栏文件浏览器，替代默认文件树 |

### 核心插件（Obsidian 内置，已启用）

Backlink, Bases, Bookmarks, Canvas, Command Palette, Daily Notes, Editor Status, File Explorer, File Recovery, Footnotes, Global Search, Graph, Markdown Importer, Note Composer, Outgoing Link, Outline, Page Preview, Properties, Random Note, Slash Command, Slides, Quick Switcher, Templates, Web Viewer, Word Count, Workspaces, Unique Note Creator

### 待安装（计划中）

| 插件 | 用途 |
|------|------|
| **Templater** | 高级模板，支持变量和日期函数 |
| **Calendar** | 日历视图，快速跳转日记 |
| **Obsidian Git** | 版本控制，自动备份到 GitHub |
| **Obsidian Web Clipper** | 网页剪藏到 `剪藏/`，支持自定义模板 |
| **Linter** | 自动规范 frontmatter 格式 |

---

## 导航方式

- **Quick Switcher**（`Cmd+O`）— 主要导航，按名字搜索，比文件树快 10 倍
- **Backlinks** — 查看哪些笔记引用了当前笔记，发现意外连接
- **`分类/` 目录** — 按类目浏览，使用 Dataview 自动聚合相关笔记
- **Graph View + Random Note** — 定期随机漫游，发现旧连接，激发灵感

---

## 快捷键设置

| 操作 | 快捷键 |
|------|--------|
| Unique Note（带时间戳新建） | 自定义（建议 `Cmd+Shift+N`） |
| Random Note（随机笔记） | 自定义 |
| Quick Switcher | `Cmd+O` |
| 插入模板（Templates） | `Cmd+Shift+T` |
| 本地图谱 | `Cmd+Shift+G`（或 Alt+G） |

---

## 参考资源

- [How I use Obsidian — Steph Ango](https://stephango.com/vault)
- [File over app — Steph Ango](https://stephango.com/file-over-app)
- [Evergreen Notes — Steph Ango](https://stephango.com/evergreen-notes)
- [40 questions to ask yourself every year](https://stephango.com/40-questions)
- [[README]] — 本 Vault 的框架设计说明
