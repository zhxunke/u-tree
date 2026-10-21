# niri 键位配置修改说明

适用环境：CachyOS + niri + Noctalia　|　配置文件：`~/.config/niri/cfg/keybinds.kdl`

## 1. 修改前准备

1. 备份原文件：

```
cp ~/.config/niri/cfg/keybinds.kdl ~/.config/niri/cfg/keybinds.kdl.bak
```

2. 编辑方式（任选其一）：

```
nano ~/.config/niri/cfg/keybinds.kdl
```

nano 常用操作：`Ctrl+O` 保存，`Ctrl+X` 退出，`Ctrl+W` 搜索。

3. 保存后 niri 会自动热重载，无需重启。检查语法：

```
niri validate
```

4. KDL 语法提示：`//` 开头是注释；每条命令以分号结尾；括号、引号要成对。

## 2. 已完成的改动（已写入新的 keybinds.kdl）

`Mod` 即 Win（Super）键。

| 需求 | 新键位 | 说明 |
| --- | --- | --- |
| 打开程序菜单（启动器） | `Win+Z` | 保留原来的 `Win+Ctrl+Return` |
| 打开会话菜单 | `Win+X` | 保留原来的 `Win+Shift+Q` |
| 打开终端 | `Win+T` | 保留原来的 `Win+Return`（Alacritty） |
| 滚轮左右切换列 | `Win+滚轮` | 向下滚为右，向上滚为左 |

为避免冲突，同时做了以下连带调整：

- 浮动/平铺窗口切换：由 `Win+T` 改为 `Win+Shift+T`。
- 滚轮切换工作区：由 `Win+滚轮` 改为 `Win+Shift+滚轮`（与原来的左右切列对调）。
- 滚轮移动列：`Win+Ctrl+滚轮` 变为左右移动列；`Win+Ctrl+Shift+滚轮` 变为把列移到上/下工作区。

## 3. 尚未写入文件、仅给出方法的改动

以下几项在对话中说明了做法，但**没有**包含在已提供的 keybinds.kdl 里，需要时自行添加。

**Win+鼠标中键关闭窗口**（保留 `Win+Q`）：在 `Mod+Q` 那行下面加一行：

```
Mod+Q                               { close-window; }
Mod+MouseMiddle                     { close-window; }
```

鼠标按键绑定需要较新版本的 niri；若 `niri validate` 报错，先执行 `sudo pacman -Syu` 更新。

**改用 Print 键截图**（可选）：

```
Print          { screenshot; }
Ctrl+Print     { screenshot-screen; }
Alt+Print      { screenshot-window; }
```

**关闭左上角热角**（滑鼠移到左上角触发总览），在 niri 配置中加入：

```
gestures {
    hot-corners {
        off
    }
}
```

用 `grep -r gestures ~/.config/niri/` 查找是否已有该段，有则直接修改，避免重复。

**关闭鼠标碰到屏幕边缘自动切换窗口**（focus-follows-mouse）：

```
grep -rn "focus-follows-mouse" ~/.config/niri/
```

通常在 `cfg/input.kdl`。

- 完全关闭：删除该行或在行首加 `//`。
- 保留焦点跟随但不在边缘滚动：改为 `focus-follows-mouse max-scroll-amount="0%"`（需较新版本 niri）。

## 4. 当前完整键位参考

**应用与 Noctalia**

- `Win+Return` / `Win+T`：终端（Alacritty）
- `Win+B`：浏览器（Firefox）
- `Win+E`：文件管理器（Nautilus）
- `Win+Z` / `Win+Ctrl+Return`：应用启动器
- `Win+S`：控制中心
- `Win+Shift+S`：Noctalia 设置
- `Win+Shift+Return`：壁纸选择器
- `Win+X` / `Win+Shift+Q`：会话菜单
- `Win+Alt+L`：锁屏
- `Win+Shift+Esc`：显示快捷键列表

**窗口与焦点**

- `Win+Q`：关闭窗口
- `Win+H/J/K/L` 或方向键：切换焦点（左/下/上/右）
- `Win+Ctrl+方向键/HJKL`：移动列或窗口
- `Win+Home/End`：跳到第一列/最后一列
- `Win+Shift+方向键`：切换显示器焦点
- `Win+Shift+Ctrl+方向键`：把列移到其他显示器
- `Win+[` / `Win+]`：并入左/右列或弹出
- `Win+,` / `Win+.`：并入当前列/弹出为独立列

**布局**

- `Win+R`：切换预设列宽（`Win+Shift+R` 反向）
- `Win+-` / `Win+=`：列宽 -10% / +10%
- `Win+Shift+-` / `Win+Shift+=`：窗口高度 -10% / +10%
- `Win+F`：最大化列；`Win+Ctrl+F`：展开到可用宽度
- `Win+Shift+F`：全屏；`Win+M`：最大化到边缘
- `Win+C`：居中列
- `Win+Shift+T`：浮动/平铺切换；`Win+Shift+V`：在浮动与平铺窗口间切换焦点
- `Win+W`：标签页显示
- `Win+O`：总览

**工作区**

- `Win+1…9`：切换工作区
- `Win+Ctrl+1…9`：把列移到对应工作区
- `Win+Tab`：回到上一个工作区
- `Win+Shift+滚轮`：上下切换工作区

**截图**

- `Ctrl+Shift+1`：选区截图
- `Ctrl+Shift+2`：整个屏幕
- `Ctrl+Shift+3`：当前窗口
- 默认存放位置通常是 `~/Pictures/Screenshots/`，可用 `grep -rn "screenshot-path" ~/.config/niri/` 确认。

**其他**

- `Win+Esc`：全屏应用抢走快捷键时用来解除
- `Win+Shift+P`：关闭显示器输出
- `Ctrl+Alt+Delete`：退出 niri（小心误按）

## 5. 注意事项

- `Win+L` 在这套配置里是“焦点右移”，不是锁屏；锁屏是 `Win+Alt+L`。
- 配置里调用 Noctalia 的命令是新版写法 `noctalia msg ...`，与网上旧教程的 `qs -c noctalia-shell ipc call ...` 不同，抄教程时注意版本。
- 新增键位不要与已有键位重复，否则 niri 会报错。
- 想临时停用某条键位，在行首加 `//` 即可，不必删除。
- 改坏了可以用备份恢复：`cp ~/.config/niri/cfg/keybinds.kdl.bak ~/.config/niri/cfg/keybinds.kdl`。
- 关于 Shorin 安装脚本：它面向纯 Arch，其 niri 分支会替换官方 niri 包，且推荐方案用的是 DMS 而非 Noctalia。在现有的 CachyOS + Noctalia 上运行容易冲突，不建议。
