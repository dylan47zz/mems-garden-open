---
categories: [tools, setups]
tags: [quartz, obsidian, digital-garden, github-pages, ci-cd]
created: 2026-03-20
status: evergreen
related: [[我的obsidian使用指南]]
publish: true
---

# 用 Quartz 把 Obsidian 笔记变成公开网站

> 只需要一个 `publish: true`，你的笔记就能自动出现在互联网上。其余的笔记？安安静静待在私有仓库里，谁也看不见。

---

## 为什么要折腾这个

写了一堆笔记，但都躺在本地硬盘上发霉。有些内容明明可以分享出去——技术总结、工具配置、踩坑记录——但又不想把整个 vault 公开（毕竟里面还有"今天摸鱼了三小时"的日记）。

**需求很简单：**
- 私有仓库保存所有笔记（包括日记和摸鱼记录）
- 只把标记了 `publish: true` 的笔记自动发布到公开网站
- push 一下就完事，不想每次手动构建

**方案：** [Quartz](https://quartz.jzhao.xyz/) + GitHub Actions + 两个仓库（一私一公）

## 最终架构

```
mems-garden (私有仓库)
  └── 笔记/xxx.md (publish: true)
        ↓ GitHub Action 自动扫描
mems-garden-open (公开仓库, Quartz)
  └── content/笔记/xxx.md
        ↓ GitHub Action 自动构建
dylan47zz.github.io/mems-garden-open 🌐
```

整个流程：**本地写笔记 → git push → 自动发布**。就这么简单。好吧，搭建过程没那么简单，但用起来是真的简单。

---

## 搭建步骤

### 第一步：创建公开仓库（Quartz）

去 GitHub 用 [jackyzha0/quartz](https://github.com/jackyzha0/quartz) 作为模板创建一个**公开**仓库，比如 `mems-garden-open`。

### 第二步：配置 Quartz

克隆公开仓库到本地，修改 `quartz.config.ts`：

```typescript
const config: QuartzConfig = {
  configuration: {
    pageTitle: "Dylan's Garden",        // 你的网站标题
    locale: "zh-CN",                     // 中文！
    baseUrl: "dylan47zz.github.io/mems-garden-open",
    analytics: null,                     // 不需要分析
    ignorePatterns: ["private", "templates", ".obsidian"],
    // ...
  },
  plugins: {
    // ...
    filters: [Plugin.ExplicitPublish()], // 🔑 关键：只发布 publish: true 的笔记
    // ...
  },
}
```

> [!important] 核心配置
> 把 `filters` 从默认的 `Plugin.RemoveDrafts()` 改成 `Plugin.ExplicitPublish()`。这样只有 frontmatter 里写了 `publish: true` 的笔记才会被构建到网站上。

创建首页 `content/index.md`：

```markdown
---
title: Dylan's Garden
publish: true
---
# Welcome to my Garden 🌱
这里是我的数字花园。
```

推送到 GitHub，开启 **Settings → Pages → Source → GitHub Actions**。

### 第三步：创建私有仓库

把你的 Obsidian vault 推送到一个**私有** GitHub 仓库：

```bash
cd mems-garden
git init && git checkout -b main
git remote add origin git@github.com:dylan47zz/mems-garden.git
git add . && git commit -m "init: vault"
git push -u origin main
```

> [!tip] .gitignore 建议
> 排除 `.obsidian/workspace.json`、`.smart-env/`、`.DS_Store` 等运行时文件。插件的 `data.json` 也建议排除。

### 第四步：配置自动同步 Action

在私有仓库创建 `.github/workflows/publish.yml`：

```yaml
name: Sync published notes to public garden

on:
  push:
    branches: [main]
    paths:
      - "**.md"
      - "附件/**"
      - ".github/workflows/publish.yml"
  workflow_dispatch:  # 支持手动触发

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/checkout@v4
        with:
          repository: dylan47zz/mems-garden-open
          token: ${{ secrets.GARDEN_PUBLISH_TOKEN }}
          path: public-garden
          ref: v4

      - name: Sync published notes
        shell: bash
        run: |
          set -e
          SOURCE="$GITHUB_WORKSPACE"
          DEST="$GITHUB_WORKSPACE/public-garden/content"

          # 清理旧内容（保留首页）
          find "$DEST" -name "*.md" -not -name "index.md" -delete 2>/dev/null || true

          count=0
          # 扫描所有 md，找 publish: true
          while IFS= read -r -d '' file; do
            if awk '
              /^---$/ { if (s==0) {s=1; next} else {exit} }
              s && /^publish:[[:space:]]*(true|True|TRUE)/ { found=1 }
              END { exit !found }
            ' "$file"; then
              rel="${file#$SOURCE/}"
              dest_file="$DEST/$rel"
              mkdir -p "$(dirname "$dest_file")"
              cp "$file" "$dest_file"
              echo "  ✅ $rel"
              ((count++)) || true
            fi
          done < <(find "$SOURCE" \
            -not -path "$SOURCE/.github/*" \
            -not -path "$SOURCE/.obsidian/*" \
            -not -path "$SOURCE/.smart-env/*" \
            -name "*.md" -print0)

          # 同步附件
          if [ -d "$SOURCE/附件" ]; then
            rm -rf "$DEST/附件"
            cp -r "$SOURCE/附件" "$DEST/附件"
          fi
          echo "✨ Synced $count notes."

      - name: Push to public garden
        working-directory: public-garden
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add content/
          if git diff --staged --quiet; then
            echo "No changes to publish."
          else
            git commit -m "sync: publish notes [$(date +'%Y-%m-%d %H:%M')]"
            git push origin v4
          fi
```

### 第五步：配置 GitHub Secret

1. 去 [GitHub Token 页面](https://github.com/settings/tokens/new) 创建 PAT，勾选 `repo` 权限
2. 去私有仓库 **Settings → Secrets → Actions → New repository secret**
3. Name: `GARDEN_PUBLISH_TOKEN`，Value: 粘贴 PAT

### 第六步：发布笔记

在任意笔记的 frontmatter 加一行：

```yaml
---
tags: [whatever]
publish: true    # ← 加这一行就行
---
```

然后 push：

```bash
cd mems-garden
git add . && git commit -m "publish: 新笔记" && git push
```

完事。等 1-2 分钟，笔记就出现在网站上了。

---

## 踩过的坑（血泪史）

### 坑 1：`((count++))` 在 `set -e` 下爆炸

Bash 的 `((count++))` 当 `count=0` 时返回值是 `0`（false），配合 `set -e` 直接退出脚本。

**解法：** `((count++)) || true`

> 这是那种"一行代码 debug 一小时"的经典 bash 时刻。

### 坑 2：`.gitignore` 阻止了 sync action 提交文件

我在公开仓库的 `.gitignore` 里加了 `content/*`，本意是本地开发时忽略同步过来的内容。结果 CI 里 `git add content/` 直接被忽略了，commit 永远是空的。

**解法：** 删掉 `.gitignore` 里的 `content/*`

### 坑 3：GitHub Actions ubuntu 没有 rsync

原本用 `rsync -a --delete` 同步附件目录，CI 直接报找不到命令。

**解法：** 改用 `rm -rf && cp -r`，朴实无华但有效。

### 坑 4：paths filter 导致 Action 不触发

Workflow 的 `paths` 设置为只监听 `.md` 文件变更，但修复 workflow 文件本身（`.yml`）时不会触发。

**解法：** paths 里加上 `.github/workflows/publish.yml`，同时加 `workflow_dispatch` 支持手动触发。

---

## 日常使用

```bash
# 写完笔记，加上 publish: true
# 然后一行命令搞定：
cd mems-garden && git add . && git commit -m "update" && git push
```

网站会在 1-2 分钟内自动更新。不需要本地构建，不需要额外操作。

## 相关链接

- [Quartz 官方文档](https://quartz.jzhao.xyz/)
- [ExplicitPublish 插件说明](https://quartz.jzhao.xyz/plugins/ExplicitPublish)
- [GitHub Pages 部署指南](https://quartz.jzhao.xyz/hosting)
