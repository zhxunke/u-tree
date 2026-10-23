# CachyOS 快照管理指南（Snapper + Btrfs Assistant）

> 前提：根分区使用 **btrfs** 文件系统。ext4 / xfs 等不支持快照。

---

## 一、安装 GUI

### 1. 确认文件系统

```bash
findmnt -no FSTYPE /          # 输出 btrfs 才能用快照
pacman -Q snapper btrfs-assistant snap-pac
snapper list-configs          # 看有没有 root 配置
```

### 2. 安装 Btrfs Assistant（推荐）

```bash
sudo pacman -S btrfs-assistant
```

装完后在应用菜单搜索 **Btrfs Assistant** 打开。功能：

- 管理 snapper 快照（创建、删除、对比、回滚）
- 修改 snapper 配置（保留数量、定时策略）
- 查看 btrfs 子卷，执行 scrub、balance 等维护

### 3. 备选：snapper-gui（AUR）

```bash
paru -S snapper-gui-git       # 或 yay -S snapper-gui-git
```

界面简陋、维护不活跃，不推荐。

### 4. 安装 / 初始化 snapper 本体

GUI 只是前端，需要 snapper 及配置：

```bash
sudo pacman -S snapper snap-pac
sudo snapper -c root create-config /    # 已有配置则不需要
```

- **snap-pac**：每次 `pacman` 操作前后自动打 pre / post 快照
- CachyOS 选 btrfs 安装时，通常已经全部配置好

---

## 二、创建快照

**GUI：** Btrfs Assistant → **Snapper** 标签 → 顶部选配置（一般是 `root`）→ **New** → 填写描述 → 确认。

**命令行：**

```bash
sudo snapper -c root create -d "装驱动前"
```

---

## 三、查看与对比

```bash
sudo snapper -c root list                      # 列出快照
sudo snapper -c root status 12..15             # 12 到 15 号之间哪些文件变了
sudo snapper -c root diff 12..15 /etc/某文件    # 查看具体内容差异
```

GUI 里选中快照后可点 **Browse / Compare**。

---

## 四、恢复 / 回滚

### 1. 只找回个别文件（最安全）

快照可直接浏览，路径：

```
/.snapshots/<编号>/snapshot/
```

把需要的文件复制出来即可。GUI 中也可用 **Restore** 恢复单个文件。

### 2. 整个系统回滚（例：回到 80 号）

**先理解编号含义：**

| 类型 | 含义 |
|------|------|
| pre | 该 pacman 操作**之前**的状态 |
| post | 该 pacman 操作**之后**的状态 |

例如 80 号是 `pacman -Rns rust` 完成后的 post 快照，回到 80 号 = 撤销 81 号（`pacman -S kitty`）及之后的所有系统改动。

**GUI 操作（能进系统时）：**

1. 在列表中选中目标快照（如 80 号）
2. 点 **Restore**
3. 恢复的子卷保持默认（`@` 根）
4. 确认，Btrfs Assistant 会先为当前状态再打一个备份快照
5. **重启**后生效

**命令行方式（不重启，逐文件撤销差异）：**

```bash
sudo snapper -c root status 80..0       # 0 代表当前系统
sudo snapper -c root undochange 80..0
```

> ⚠️ `undochange` 涉及软件包时，pacman 数据库与实际文件可能不一致。
> 事后建议执行 `sudo pacman -Syu` 并用 `pacman -Qk` 检查。
> **整体回滚（Restore + 重启）更干净。**

### 3. 进不了系统时

- **Limine 引导**：配合 `limine-snapper-sync`，开机菜单可直接选快照启动，进去后再正式回滚
- **GRUB 引导**：安装 `grub-btrfs`，GRUB 菜单里会出现快照条目

---

## 五、删除快照

### 命令行

```bash
sudo snapper -c root list                  # 先看编号
sudo snapper -c root delete 77             # 删单个
sudo snapper -c root delete 70-80          # 删一段
sudo snapper -c root delete 70-80 85 90    # 多个 / 多段一起删
sudo snapper -c root delete 1-75           # 只保留最近的
```

- snap-pac 的快照是 pre / post 成对的，建议**成对一起删**（如 `79-80`）
- 编号 0 是"当前系统"，不能删；正在使用的默认子卷快照也删不掉

### GUI

1. **Ctrl + 点击** 多选，或点一个后 **Shift + 点击** 另一个批量选择
2. 点 **Delete**，确认

### 释放空间

btrfs 删快照后台异步回收空间，想等它完成：

```bash
sudo btrfs subvolume sync /
```

---

## 六、自动清理设置

编辑 `/etc/snapper/configs/root`：

```ini
NUMBER_LIMIT="10"             # pacman 类快照保留数量
NUMBER_LIMIT_IMPORTANT="5"    # 重要快照保留数量
TIMELINE_CREATE="no"          # 关闭定时快照
```

也可以在 Btrfs Assistant 的 **Snapper Settings** 里图形化修改。改完后 snapper 的清理定时器会按新规则自动清理。

---

## 七、注意事项

- **快照不是备份**：与原数据在同一块硬盘，硬盘损坏则快照一并丢失，重要数据需另外备份。
- **`/home` 默认不在快照范围内**：回滚系统不会动个人文件。想给 `/home` 做快照：

  ```bash
  sudo snapper -c home create-config /home
  ```

- **回滚的代价**：目标快照之后安装的软件和修改的系统配置都会消失。
- **小问题不必回滚**：例如只是想卸载某个软件，直接 `sudo pacman -Rns <包名>` 更简单。

---

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 创建 | `sudo snapper -c root create -d "描述"` |
| 列表 | `sudo snapper -c root list` |
| 对比 | `sudo snapper -c root status A..B` |
| 撤销差异 | `sudo snapper -c root undochange A..0` |
| 删除 | `sudo snapper -c root delete 编号` |
| 同步回收空间 | `sudo btrfs subvolume sync /` |
