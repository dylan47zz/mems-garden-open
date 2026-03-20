---
categories: [tools, setups]
tags: [obsidian, sync, android, syncthing, mobile]
created: 2026-03-20
status: evergreen
related: [[Obsidian-Git多端同步配置], [我的obsidian使用指南]]
publish: true
---

# 用 Syncthing 同步 Obsidian 到安卓手机

> 不需要 GitHub，不需要 Termux，装两个 app，几分钟搞定。桌面继续用 git 发布，手机用 Syncthing 同步，互不干扰。

---

## 原理

Syncthing 是一个 P2P 文件同步工具，直接在设备间传输，不经过任何云服务器。

```
Mac（mems-garden 目录）
    ↕ Syncthing（局域网直连 / 跨网络中继）
Android（Obsidian vault 目录）
```

桌面端的 git push 发布流程完全不变，Syncthing 只管文件同步。

---

## 安装

### Mac 端

去 [Syncthing 官网](https://syncthing.net/downloads/) 下载 macOS 版，或用 Homebrew：

```bash
brew install --cask syncthing
```

安装后启动，浏览器会自动打开 `http://127.0.0.1:8384`，这是 Syncthing 的 Web UI。

> [!tip] 开机自启
> Mac 端建议用 [Syncthing for macOS](https://github.com/syncthing/syncthing-macos) 图形版，带菜单栏图标，支持开机自启，比命令行版方便。

### Android 端

在 Google Play 或 F-Droid 安装 **Syncthing**（官方 app，包名 `com.nutomic.syncthingandroid`）。

> [!warning] 注意权限
> 首次启动需要授予「存储」权限，否则无法读写 Obsidian 的 vault 目录。

---

## 配置步骤

### 第一步：配对两台设备

**在 Android 端**，打开 Syncthing app：
1. 进入「设备」标签页
2. 点右上角 `+`，扫描 Mac 端的设备 ID 二维码

**在 Mac 端**，打开 Web UI（`http://127.0.0.1:8384`）：
1. 点右上角菜单 → **Actions → Show ID**，会显示二维码
2. 等待 Android 发来配对请求，点「添加设备」确认

配对成功后，两端设备列表里会出现对方，状态显示「已连接」。

---

### 第二步：共享 mems-garden 文件夹

**在 Mac 端 Web UI**：

1. 左侧「文件夹」区域 → 找到 `mems-garden` 文件夹（如果没有就点「添加文件夹」）
2. 填写：
   - **文件夹 ID**：随意，比如 `mems-garden`
   - **文件夹路径**：`/Users/dylan/my-all/notes/mems-garden`
3. 切换到「共享」标签页，勾选刚才添加的 Android 设备
4. 保存

**在 Android 端**，会弹出一条通知：「Mac 想与你共享文件夹 mems-garden」：

1. 点「添加」
2. 设置本地路径，比如 `/storage/emulated/0/Documents/mems-garden`
3. 确认

> [!tip] Android 路径建议
> 建议放在 `/storage/emulated/0/Documents/` 下，Obsidian 能直接打开这个路径的 vault。

---

### 第三步：Obsidian 打开 vault

等首次同步完成（进度条跑完），在 Android Obsidian 里：

1. 打开 Obsidian → **打开另一个 vault**
2. 选择「打开本地文件夹中的 vault」
3. 找到刚才 Syncthing 同步的路径，比如 `/storage/emulated/0/Documents/mems-garden`
4. 信任并打开

所有笔记就都出现了。

---

## 日常使用

同步是**自动、实时**的，不需要手动触发。

```
Mac 保存文件
    ↓（几秒内）
Android Syncthing 检测到变更，自动同步
    ↓
Obsidian 刷新，看到最新内容 ✅
```

### 手动触发同步

如果两端不在同一网络导致同步延迟，打开 Syncthing app，下拉刷新即可。

---

## 与 git 发布流程的关系

Syncthing 同步的是**工作目录中的文件**，包括 `.git` 文件夹。这没有影响，Syncthing 会正确同步它。

| 操作 | 用什么 |
|------|--------|
| 手机 ↔ Mac 文件同步 | Syncthing |
| 发布到 Quartz 数字花园 | git push → GitHub Actions |

两者完全独立，不冲突。

> [!note] 要不要在手机上 commit？
> 完全不需要。手机只是读写文件，git 的事情交给 Mac 就够了。Obsidian Git 插件在手机上可以不装。

---

## 跨网络同步说明

| 场景 | 同步方式 | 速度 |
|------|----------|------|
| 同一 WiFi | 直接 P2P | 极快 |
| 不同网络（4G/不同 WiFi） | 公共中继服务器 | 较慢，但对笔记够用 |

笔记都是文本文件，体积小，跨网络中继也能在几秒到十几秒内同步完成。

---

## 踩坑记录

### Android 找不到 vault 目录
Syncthing 没有存储权限，去「设置 → 应用 → Syncthing → 权限」手动开启。

### 同步状态一直显示「失步」
两端时间不同步可能导致问题，检查手机时间是否设置为自动同步。

### Mac 端文件夹没出现在 Syncthing
直接在 Web UI 手动添加文件夹，路径填绝对路径。

### 冲突文件（`.sync-conflict-...`）
两端同时编辑了同一个文件，Syncthing 会保留两份。在 Mac 上手动合并后删掉冲突文件即可。出现概率很低，因为手机和电脑一般不会同时编辑同一笔记。

---

## 相关链接

- [Syncthing 官网](https://syncthing.net/)
- [Syncthing for macOS（图形客户端）](https://github.com/syncthing/syncthing-macos)
- [Syncthing Android（Google Play）](https://play.google.com/store/apps/details?id=com.nutomic.syncthingandroid)
