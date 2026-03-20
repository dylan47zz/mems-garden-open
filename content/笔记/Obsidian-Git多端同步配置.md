---
categories: [tools, setups]
tags: [obsidian, git, sync, mobile, multi-device]
created: 2026-03-20
status: evergreen
related: [[我的obsidian使用指南], [用Quartz搭建数字花园]]
publish: true
---

# 用 Obsidian Git 插件多端自动备份同步

> 一份配置，所有设备自动同步。Mac 上写完，手机上看到。偶尔崩溃，但瑕不掩瑜。

---

## 前置准备

- GitHub 私有仓库（已有：`git@github.com:dylan47zz/mems-garden.git`）
- Mac 已配置 SSH key 并推送过仓库
- 手机需要 GitHub PAT（Personal Access Token）

### 创建 PAT（手机端必须）

去 https://github.com/settings/tokens/new 创建：
- **Note**: `obsidian-mobile`
- **Expiration**: 1 年
- **Scopes**: 勾选 `repo`

复制 token，安全保存（只显示一次，手机端会用到）。

---

## Mac 配置

Mac 使用系统原生 Git + SSH，最稳定。

### 第一步：安装插件

Obsidian → 设置 → 第三方插件 → 浏览社区插件 → 搜索 `Obsidian Git` → 安装并启用

### 第二步：认证（macOS Keychain）

打开终端，运行一次：

```bash
git config --global credential.helper osxkeychain
```

如果已经用 SSH（推荐），则不需要额外配置，Obsidian Git 会直接调用系统 Git。

### 第三步：插件设置

进入 Obsidian → 设置 → Obsidian Git，关键配置：

| 设置项 | 推荐值 | 说明 |
|--------|--------|------|
| Vault backup interval | `10` | 每 10 分钟自动 commit + push |
| Auto pull interval | `10` | 每 10 分钟自动 pull |
| Pull updates on startup | ✅ 开启 | 启动时先 pull，避免冲突 |
| Commit message | `vault backup: {{date}}` | 自动加时间戳 |
| Pull strategy | `rebase` | 减少 merge commit |

> [!tip] 如果已有本地仓库
> 直接在 vault 目录打开 Obsidian，插件会自动识别已有的 `.git` 目录，无需重新 clone。

---

## Android 配置

> [!warning] 注意
> Android 端使用的是 JavaScript 实现的 Git（isomorphic-git），**不支持 SSH**，只支持 HTTPS + PAT。仓库太大可能崩溃或卡死，属于正常现象。

### 第一步：安装 Obsidian

安装 [Obsidian for Android](https://play.google.com/store/apps/details?id=md.obsidian)，创建一个**空的新 vault**（先别放任何文件）。

### 第二步：安装 Git 插件

设置 → 第三方插件 → 浏览社区插件 → 搜索 `Obsidian Git` → 安装并启用

### 第三步：填入认证信息

进入插件设置 → **Authentication/Commit Author** 区域：

| 字段 | 填入内容 |
|------|----------|
| Username | `dylan47zz`（GitHub 用户名） |
| Password | 粘贴之前创建的 PAT |
| Author name | `dylan47` |
| Author email | `dylan47zz@users.noreply.github.com` |

### 第四步：Clone 仓库

打开命令面板（右上角 `...` → 命令）→ 搜索 `Git: Clone existing remote repo`

填入仓库地址（**必须用 HTTPS 格式，末尾加 `.git`**）：
```
https://github.com/dylan47zz/mems-garden.git
```

等待 clone 完成（会有进度弹窗），完成后提示重启 Obsidian。重启后所有笔记就都出现了。

### 第五步：插件设置（手机）

| 设置项 | 推荐值 |
|--------|--------|
| Vault backup interval | `30`（手机耗电，频率低一点） |
| Pull updates on startup | ✅ 开启 |
| Disable on this device | ❌ 保持关闭 |

> [!tip] 手机使用建议
> - 不要在手机上写大量内容后立刻换 Mac 写，容易冲突
> - 手动同步：命令面板 → `Git: Commit-and-sync`
> - 仓库太大导致 clone 崩溃？试试先删掉 `附件/` 里的大文件再 push

---

## 多端同步的正确姿势

```
写完内容
    ↓
自动/手动 Commit-and-sync（push 到 GitHub）
    ↓
切换设备打开 Obsidian
    ↓
自动 pull（启动时）或手动 pull
    ↓
看到最新内容 ✅
```

### 避免冲突的原则

1. **切换设备前先同步**：离开一台设备时手动触发一次 commit-and-sync
2. **启动时自动 pull**：已配置，无需操心
3. **不要同时在两台设备编辑同一个文件**：git 不是实时协同工具

> [!warning] 冲突怎么办？
> 插件会弹出提示。一般选择 **rebase** 策略可以自动解决大部分冲突。真的冲突了，打开命令面板 → `Git: Open source control view` 手动处理。

---

## 常用命令（命令面板）

| 命令 | 作用 |
|------|------|
| `Git: Commit-and-sync` | 手动触发：commit + pull + push |
| `Git: Pull` | 只 pull |
| `Git: Push` | 只 push |
| `Git: Open source control view` | 查看文件变更状态 |
| `Git: Open history view` | 查看 commit 历史 |

---

## 踩坑记录

### Android clone 卡死
大仓库 clone 可能需要几分钟，期间不要操作 Obsidian。真的卡住了就强杀重启，重试。

### 提示 "remote: Invalid username or password"
PAT 填错了，或者 PAT 过期了。重新去 GitHub 生成一个，在插件设置里更新。

### Mac 端自动同步没触发
检查 Obsidian Git 插件设置里 `Vault backup interval` 是否大于 0，以及 vault 是否真的有 git remote。

### 手机和 Mac 产生冲突
- 冲突文件会有 `<<<<<<` 标记
- 用 Mac 打开，手动合并内容，删掉冲突标记，再 commit-and-sync

---

## 相关链接

- [Obsidian Git 插件仓库](https://github.com/Vinzent03/obsidian-git)
- [官方文档](https://publish.obsidian.md/git-doc)
- [Authentication 指南](https://publish.obsidian.md/git-doc/Authentication)
