# 文件管理器「右键 → 打开方式 → Neovim」无反应 排查与修复记录

- 日期：2026-10-22
- 系统：CachyOS Linux（Arch 系，滚动更新）
- 桌面环境：niri（Wayland）+ Nautilus 51.1
- 编辑器：Neovim v0.12.6 + LazyVim（配置位于 `~/.config/nvim`）
- **最终状态：已根治。所有 `Terminal=true` 的程序（Neovim、btop、htop 等）现在都能从图形界面正常启动。**

---

## 一、现象

在文件管理器中右键点击文件 → 「打开方式」→ 选择 **Neovim**，没有任何反应：
不弹窗、不报错、也不出现新窗口。

---

## 二、根本原因

**与 LazyVim、Neovim 本身无关**，是桌面启动项（`.desktop`）与系统默认终端缺失导致的。

`nvim.desktop` 的原始内容包含：

```ini
Exec=nvim %F
Terminal=true
```

`Terminal=true` 表示「该程序是命令行程序，请帮我启动一个终端来承载它」。
此时 GLib 会按以下顺序寻找终端启动器：

1. `xdg-terminal-exec`
2. 环境变量 `$TERMINAL`
3. 硬编码回退列表（gnome-terminal、kgx、konsole、xfce4-terminal、xterm 等）

实际检查结果：

| 检查项 | 结果 |
| --- | --- |
| `xdg-terminal-exec` | 未安装 |
| `$TERMINAL` 环境变量 | 空 |
| gnome-terminal / xterm / konsole 等 | 均未安装 |
| 实际存在的终端 | 只有 `alacritty`（未被登记为默认终端） |

因此 GLib 找不到任何可用的终端，直接放弃拉起进程 —— 表现为「点了没反应」。

### 直接验证命令

```bash
gio launch ~/.local/share/applications/nvim.desktop
```

输出：

```
gio: 无法启动应用程序"/home/xunke/.local/share/applications/nvim.desktop": 无法找到应用程序需要的终端
```

另外发现该用户级 desktop 文件里带有 `NoDisplay=true`，这会让条目**不出现在「打开方式」候选列表中**，也是一个需要处理的点。

---

## 三、修复过程

### 阶段一：用户级临时方案（已验证，后已还原）

核心思路：不再依赖「系统帮忙找终端」，而是在启动项里显式用 Alacritty 承载 Neovim。

修改 `~/.local/share/applications/nvim.desktop`（先备份）：

| 键 | 修改前 | 修改后 |
| --- | --- | --- |
| `Exec` | `nvim %F` | `alacritty --class nvim -e nvim %F` |
| `Terminal` | `true` | `false` |
| `StartupWMClass` | （无） | `nvim`（新增） |
| `NoDisplay` | `true` | 已删除 |
| `MimeType` | 基础 C/脚本类 | 增补 `text/x-lua`、`text/x-python`、`text/x-rust`、`text/x-go`、`application/json`、`application/toml`、`application/x-yaml`、`text/markdown` |

此方案有效，但只解决 Neovim 一个程序，btop/htop 等仍然点不开，属于治标。

### 阶段二：根治 —— 安装标准终端启动器 `xdg-terminal-exec`

`xdg-terminal-exec` 是 XDG「默认终端执行规范」的参考实现，GLib 会优先调用它。

> **说明：** 官方软件包在 Arch 的 `extra` 仓库（`xdg-terminal-exec 0.14.3-2`），
> 但本机 `sudo` 需要密码、且未安装 polkit 认证代理（图形授权弹框不可用），
> 因此采用**免 root 方式**：把上游官方脚本安装到已在 `PATH` 中的 `~/.local/bin`，
> 功能与软件包一致，区别仅是不受 pacman 管理、升级需手动重复以下步骤。
>
> 若日后配置好 sudo，可用一条命令改成软件包版本（需先删除 `~/.local/bin/xdg-terminal-exec`，避免遮蔽系统版本）：
>
> ```bash
> sudo pacman -S xdg-terminal-exec
> rm ~/.local/bin/xdg-terminal-exec
> ```

#### 2.1 安装

```bash
git clone --depth 1 https://github.com/Vladimir-csp/xdg-terminal-exec /tmp/xte

# 主脚本（~/.local/bin 已在 PATH 中）
install -Dpm755 /tmp/xte/xdg-terminal-exec ~/.local/bin/xdg-terminal-exec

# 上游默认规则（各终端的 execarg 默认值等）
install -Dpm644 /tmp/xte/xdg-terminals.list \
  ~/.local/share/xdg-terminal-exec/xdg-terminals.list
```

#### 2.2 指定默认终端

配置文件是 **`xdg-terminals.list`**（不是 `xdg-terminal-exec.conf`），
格式为按优先级排列的 desktop entry ID：

`~/.config/xdg-terminals.list`：

```ini
# 默认终端（优先级从高到低），xdg-terminal-exec 会按顺序取第一个可用的
Alacritty.desktop
```

> 说明：`Alacritty.desktop` 的 `Categories` 含 `System;TerminalEmulator;`，
> 满足规范的必要条件，因此能被正确识别。

#### 2.3 还原临时方案，回到系统原始条目

删掉用户级覆盖，改用 `/usr/share/applications/nvim.desktop`（该文件本身没有 `NoDisplay`）：

```bash
mv ~/.local/share/applications/nvim.desktop \
   ~/.local/share/applications/nvim.desktop.alacritty.bak
update-desktop-database ~/.local/share/applications
```

现在生效的条目恢复为原始写法：

```ini
Exec=nvim %F
Terminal=true
```

---

## 四、验证

### 4.1 终端选择是否正确

```bash
$ xdg-terminal-exec --print-id
Alacritty.desktop

$ xdg-terminal-exec --print-cmd nvim /etc/os-release
alacritty
-e
nvim
/etc/os-release

$ xdg-terminal-exec --print-cmd btop
alacritty
-e
btop
```

`--print-cmd` 只打印最终命令行、不真正启动窗口，可见 Alacritty 被正确选中且参数拼接正确。

### 4.2 完整的 GLib 启动链路（含 `Terminal=true`）

用与 `nvim.desktop` 完全同构的临时条目（仅把 `Exec` 换成一个写日志的脚本），走真实启动路径：

```bash
timeout 20 gio launch /tmp/nvim-verify.desktop /etc/os-release
# launch-exit=0
# ok args: /etc/os-release
```

结论：`gio` → `xdg-terminal-exec` → `alacritty -e <命令>` 整条链路打通，文件参数正确传递。

### 4.3 Neovim 本体

```bash
nvim --headless +'lua print("nvim ok")' +q   # nvim ok, exit=0
```

---

## 五、当前状态

### 新增/修改的文件

| 路径 | 说明 |
| --- | --- |
| `~/.local/bin/xdg-terminal-exec` | 终端启动器主脚本（755） |
| `~/.local/share/xdg-terminal-exec/xdg-terminals.list` | 上游默认规则（644） |
| `~/.config/xdg-terminals.list` | 指定默认终端为 `Alacritty.desktop` |
| `~/.local/share/applications/nvim.desktop.alacritty.bak` | 阶段一临时方案的备份（含 `alacritty -e nvim` 写法、补充的 MimeType） |
| `~/.local/share/applications/nvim.desktop.bak` | 最初的桌面条目原文（含 `NoDisplay=true`） |

两个备份文件都以 `.bak` 结尾，不会被 desktop 数据库读取，不产生任何影响，可随时删除。

### 已解决的问题

- 「右键 → 打开方式 → Neovim」可正常打开文件
- 所有 `Terminal=true` 的程序（btop、htop 等）同样恢复正常
- Neovim 条目不再带 `NoDisplay=true`，能正常出现在「打开方式」及应用列表中

> 若 Nautilus 仍显示旧状态，执行 `nautilus -q` 关闭后重新打开即可（GLib 侧无需重启）。

---

## 六、回滚方式

### 回滚终端启动器（恢复到修复前状态）

```bash
rm -f ~/.local/bin/xdg-terminal-exec
rm -rf ~/.local/share/xdg-terminal-exec
rm -f ~/.config/xdg-terminals.list
```

### 回滚 Neovim 桌面条目

如需恢复最初的原文（注意会把 `NoDisplay=true` 一起带回来，Neovim 将重新从「打开方式」列表消失）：

```bash
cp ~/.local/share/applications/nvim.desktop.bak \
   ~/.local/share/applications/nvim.desktop
update-desktop-database ~/.local/share/applications
```

---

## 七、备注与可选项

### 7.1 更正：配置文件位置

早期文档曾写成：

```bash
# ❌ 错误写法
mkdir -p ~/.config/xdg-terminal-exec
printf 'default=alacritty\n' > ~/.config/xdg-terminal-exec/xdg-terminal-exec.conf
```

该路径/键名属于其他实现，**对本实现（Vladimir-csp 参考实现）无效**。
正确做法是本文 2.2 节：`~/.config/xdg-terminals.list` 中写 `Alacritty.desktop`。

### 7.2 把 Neovim 设为文本文件默认程序

当前 `text/plain` 的默认程序仍是 `com.microsoft.VSCode.desktop`。如需更改：

```bash
xdg-mime default nvim.desktop text/plain
```

### 7.3 man 手册（可选）

上游 man 页需要 `scdoc` 编译，本次未安装（不影响功能）：

```bash
sudo pacman -S scdoc        # 或手动使用 scdoc
cd /path/to/xdg-terminal-exec && make && make install-man
```

### 7.4 升级脚本版本

免 root 安装需手动更新，重复 2.1 节的 `git clone` + `install` 即可。
若已安装 pacman 版本，则随系统正常升级。
