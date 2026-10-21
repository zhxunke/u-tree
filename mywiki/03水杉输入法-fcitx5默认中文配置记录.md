# fcitx5 + 水杉输入法：全局默认中文配置记录

- 日期：2026-10-21
- 系统：CachyOS（rolling），niri（Wayland）
- 相关版本：
  - `fcitx5-shorin-patched-git 5.1.22.r5.g4146a2a7-1`
  - `msime-bin 0.12.1-1`（水杉输入法）
  - `librime 1:1.17.0-6.1`（仅 Rime 引擎保留，雾凇方案此前已卸载）

## 1. 目标

让水杉成为全局默认的中文输入法：

- 新开窗口 / 切换窗口后，直接打字就是中文候选；
- 不再需要先 `Ctrl+空格` 激活输入法、再 `Shift` 切到中文；
- `Shift` 仍然可以在水杉内部切换中/英，并且保留「中/英」提示浮层。

## 2. 先厘清两层状态

在 Wayland + fcitx5 下，「能不能输入中文」其实分两层，之前的困扰来自两层混在一起：

| 层 | 状态 | 由谁控制 | 表现 |
|---|---|---|---|
| 第一层 | 输入法框架 激活 / 非激活 | fcitx5 | 非激活时按键直接走键盘布局（`keyboard-us`），完全不经过输入法 |
| 第二层 | 中 / 英 | 水杉自身 | 框架激活后，水杉内部还有中文模式和英文模式 |

原来的流程：

```text
Ctrl+空格  →  第一层：fcitx5 从非激活变为激活（当前 IM = msime）
Shift      →  第二层：水杉从英文模式切到中文模式
```

## 3. 关键文件

| 文件 | 作用 |
|---|---|
| `~/.config/fcitx5/config` | fcitx5 全局配置（本次新建） |
| `~/.config/fcitx5/profile` | 输入法分组与 `DefaultIM=msime`（本次未改） |
| `~/.config/msime-client/preferences.json` | 水杉偏好设置的权威副本，带 `revision` |
| `~/.config/msime-client/runtime-options.json` | fcitx5 插件实际读取的运行时配置快照 |
| `~/.local/share/app.msime.linux/preferences.json` | 水杉桌面设置程序自己的副本 |
| `/run/user/1000/msime-client/input-status.json` | 水杉运行状态：`active`、`label`（中/英）、`scheme` |

## 4. 诊断过程与用到的命令

### 4.1 查看 fcitx5 当前状态

```bash
# 0 = 关闭输入法，1 = 非激活（keyboard-us），2 = 激活
fcitx5-remote

# 当前输入法名称
fcitx5-remote -n
```

一开始观察到：

```text
state=1 im=keyboard-us      # 切到某个窗口后是“非激活”
```

### 4.2 读取 fcitx5 全局配置的运行时值

```bash
busctl --user call org.fcitx.Fcitx5 /controller \
  org.fcitx.Fcitx.Controller1 GetConfig s 'fcitx://config/global'
```

关键默认值（未修改前）：

```text
ActiveByDefault = False        # 新窗口默认不激活输入法
ShareInputState = No           # 输入法状态每个窗口各自独立
TriggerKeys     = Control+space  # 被 fcitx5 用来 激活/关闭 输入法
AltTriggerKeys  = Shift_L        # fcitx5 抢走了 Shift 作为“临时切换输入法”
```

### 4.3 查看水杉自身状态

```bash
cat /run/user/1000/msime-client/input-status.json
# {"active":true,"label":"中","scheme":"quanpin"}

msime config get default_ime_mode
# chinese

msime config
```

水杉偏好里：

```text
default_ime_mode = chinese   # 本来就是默认中文
ime_mode_scope   = app       # 中英模式按应用分别记忆
```

### 4.4 观察窗口切换时的状态

```bash
niri msg --json windows
niri msg action focus-window --id <窗口ID>

# 打开一个新的临时窗口做测试
niri msg action spawn -- alacritty -e sleep 5
```

实测结果：

| 配置组合 | 新开窗口后的状态 |
|---|---|
| `ActiveByDefault=True` + `ShareInputState=All` | `state=1`（继承了共享的“关闭”状态）❌ |
| `ActiveByDefault=True` + `ShareInputState=No` | `state=2`、`im=msime`、水杉“中” ✅ |

结论：`ShareInputState=All` 会把「输入法被关掉」这个状态共享给所有窗口，
连新窗口也继承，因此不适合“永远默认中文”的目标。

### 4.5 中/英提示浮层消失的原因

水杉的提示浮层由 fcitx5 插件里的 `FcitxState::showInputModeHud()` 绘制，
底层是 wlr-layer-shell 的 `ModeBadgeWaylandSurface`。

它只在**语言状态真正发生变化**时出现。最初为了防误触，把水杉的
`Ctrl+空格` 中英切换关掉了，所以按 `Ctrl+空格` 时既没有切换、也没有提示。
后来恢复为开启，提示即恢复。

## 5. 最终修改

### 5.1 fcitx5 全局配置

新建 `~/.config/fcitx5/config`：

```ini
[Behavior]
ActiveByDefault=True
ShareInputState=No

[Hotkey/TriggerKeys]
0=Control+Shift+space

[Hotkey/AltTriggerKeys]
```

说明：

| 配置项 | 值 | 作用 |
|---|---|---|
| `ActiveByDefault` | `True` | 每个新窗口/新输入上下文默认激活输入法 |
| `ShareInputState` | `No` | 各窗口状态独立，避免“关闭”状态传染；每个新窗口仍会默认激活 |
| `TriggerKeys` | `Control+Shift+space` | 只作为应急开关输入法，不再占用 `Ctrl+空格` |
| `AltTriggerKeys` | 空 | 不再让 fcitx5 抢 `Shift`，把中/英切换完全交给水杉 |

注意：在 fcitx5 的配置里，空 section（例如 `[Hotkey/AltTriggerKeys]`）表示空列表。
可通过 `GetConfig` 验证，空列表会显示成 `s ""`。

### 5.2 水杉偏好设置

需要修改的键：

```text
ime_mode_scope                 = global   # 中英模式全局共享，而不是按应用记忆
default_ime_mode               = chinese  # 默认中文（原本就是）
input_mode_hud                 = true     # 保留中/英提示浮层
keybindings.switch_language_shift      = true  # Shift 切换中/英
keybindings.switch_language_ctrl_space = true  # Ctrl+空格 切换中/英（并弹提示）
```

涉及的三个文件：

```text
~/.config/msime-client/preferences.json                 # 带 revision，需要 +1
~/.config/msime-client/runtime-options.json             # fcitx5 插件读取
~/.local/share/app.msime.linux/preferences.json         # 桌面设置程序副本
```

实际使用的修改脚本（保留其余字段、原子写入、权限 600）：

```python
import json, os, tempfile

paths = [
    '/home/xunke/.config/msime-client/preferences.json',
    '/home/xunke/.config/msime-client/runtime-options.json',
    '/home/xunke/.local/share/app.msime.linux/preferences.json',
]

for path in paths:
    with open(path) as f:
        d = json.load(f)

    p = d.setdefault('preferences', {})
    p['ime_mode_scope'] = 'global'
    p['default_ime_mode'] = 'chinese'

    kb = p.setdefault('keybindings', {})
    kb['switch_language_shift'] = True
    kb['switch_language_ctrl_space'] = True

    # preferences.json 有 revision，用于让设置程序识别变化
    if 'revision' in d:
        d['revision'] += 1

    fd, tmp = tempfile.mkstemp(dir=os.path.dirname(path))
    with os.fdopen(fd, 'w') as f:
        json.dump(d, f, ensure_ascii=False, indent=2)
        f.write('\n')

    os.chmod(tmp, 0o600)
    os.replace(tmp, path)
    print(path, 'ok, revision =', d.get('revision'))
```

### 5.3 修改前先关掉水杉设置程序

设置程序运行时可能在退出/改设置时用内存里的旧配置回写文件，因此修改前先关闭：

```bash
# 关闭设置窗口（niri）
niri msg action close-window --id <设置窗口ID>

# 确认进程是否还在，若还在则结束
pgrep -af msime-linux-desktop
kill -TERM <PID>        # 必要时 kill -KILL <PID>
```

### 5.4 重启 fcitx5 使配置生效

```bash
# -r 替换现有实例，-d 后台运行
setsid fcitx5 -r -d
```

如果是修改 fcitx5 的全局配置，也可以先尝试热重载：

```bash
fcitx5-remote -r
```

但 `ActiveByDefault` 这类行为选项建议直接重启 fcitx5 更稳妥。

### 5.5 把已经开着的窗口也激活一遍

`ShareInputState=No` 下，修改前就已存在的窗口可能仍是旧的“非激活”状态，
逐个聚焦并激活：

```bash
for id in $(niri msg --json windows | \
  python3 -c "import sys,json;print(' '.join(str(w['id']) for w in json.load(sys.stdin)))"); do
    niri msg action focus-window --id "$id"
    sleep 0.3
    fcitx5-remote -o
    sleep 0.2
done
```

## 6. 验证

### 6.1 验证 fcitx5 配置已加载

```bash
busctl --user call org.fcitx.Fcitx5 /controller \
  org.fcitx.Fcitx.Controller1 GetConfig s 'fcitx://config/global'
```

应看到：

```text
"ActiveByDefault" s "True"
"ShareInputState" s "No"
"TriggerKeys" a{sv} 1 "0" s "Control+Shift+space"
"AltTriggerKeys" s ""
```

### 6.2 验证新窗口默认中文

```bash
# 打开一个 5 秒后自动关闭的临时终端
niri msg action spawn -- alacritty -e sleep 5
sleep 1.5

fcitx5-remote            # 期望输出 2（激活）
fcitx5-remote -n         # 期望输出 msime
cat /run/user/1000/msime-client/input-status.json
# 期望 {"active":true,"label":"中",...}
```

### 6.3 验证水杉偏好

```bash
python3 -c "
import json
d=json.load(open('/home/xunke/.config/msime-client/preferences.json'))
p=d['preferences']; k=p.get('keybindings',{})
print('scope      =', p.get('ime_mode_scope'))
print('default    =', p.get('default_ime_mode'))
print('hud        =', p.get('input_mode_hud'))
print('shift      =', k.get('switch_language_shift'))
print('ctrl_space =', k.get('switch_language_ctrl_space'))
"
```

期望：

```text
scope      = global
default    = chinese
hud        = True
shift      = True
ctrl_space = True
```

## 7. 最终按键行为

| 操作 | 结果 |
|---|---|
| 新开窗口直接打字 | 水杉中文候选（输入法已默认激活） |
| `Shift` | 水杉内 中 ↔ 英，弹出中/英提示 |
| `Ctrl+空格` | 水杉内 中 ↔ 英，并弹出中/英提示（不再关闭 fcitx5） |
| `Ctrl+Shift+空格` | 应急：开/关整个输入法 |
| `Super+空格` | fcitx5 输入法分组切换（只有一组，通常无感） |

## 8. 回滚方法

### 8.1 fcitx5

把 `~/.config/fcitx5/config` 改为原默认行为，或直接删除该文件：

```bash
rm ~/.config/fcitx5/config
fcitx5 -r -d
```

原默认值等价于：

```ini
[Behavior]
ActiveByDefault=False
ShareInputState=No

[Hotkey/TriggerKeys]
0=Control+space

[Hotkey/AltTriggerKeys]
0=Shift_L
```

### 8.2 水杉

把 `ime_mode_scope` 改回 `app`，并重新打开设置程序修改即可：

```text
ime_mode_scope = app
```

修改前生成的临时备份（可能已被系统清理）：

```text
/tmp/opencode/fcitx5-profile.bak
/tmp/opencode/msime-preferences.bak
/tmp/opencode/msime-runtime-options.bak
```

## 9. 备注

- 本次没有改动 fcitx5 的输入法列表，`~/.config/fcitx5/profile` 中仍然是
  `keyboard-us`、`rime`、`msime` 三项，`DefaultIM=msime`。
- 水杉设置窗口在修改配置时被关闭过，重新从应用菜单打开即可。
- 如果以后水杉升级重置了偏好，重点检查 `ime_mode_scope`、`input_mode_hud`
  和两个语言切换快捷键是否还在。
- 如果某个应用（如全屏游戏、密码框）不遵循默认激活，这是应用/协议本身的行为，
  可用 `Ctrl+Shift+空格` 临时处理。
- 雾凇拼音（`rime-ice-git`）在本文档之前已经卸载，Rime 用户目录中只剩
  明月拼音（`luna_pinyin`）的用户词库，与本文的默认中文配置无关。

## 10. 补充：Typora 无法输入中文（XWayland）

### 10.1 现象

其他应用（Firefox、Alacritty、微信等）中文输入正常，唯独 Typora 打不出中文候选。

### 10.2 原因

| 项目 | 情况 |
|---|---|
| Typora 版本 | `typora-free 0.11.18-4` |
| 运行时 | 老版 Electron，不支持 Wayland，自动跑在 **XWayland** 里 |
| 输入法通道 | Wayland 应用走 text-input 协议；XWayland 应用走 **XIM** |
| 缺失的环境变量 | `XMODIFIERS=@im=fcitx` 没有设置 |

也就是说：

- 原生 Wayland 应用：fcitx5 通过 Wayland text-input 直接工作，不需要环境变量；
- Typora（XWayland）：必须通过 XIM 连接 fcitx5，而 XIM 依赖 `XMODIFIERS`。

niri 会话里原本没有设置 `XMODIFIERS`，所以只有 Typora 这类 XWayland 应用无法输入中文。

### 10.3 修改一：给 niri 会话加环境变量

编辑 `~/.config/niri/cfg/misc.kdl`，在已有的 `environment { ... }` 块中加入：

```kdl
environment {
    // ...原有内容保持不变...
    XMODIFIERS "@im=fcitx"
}
```

校验并热加载：

```bash
niri validate
niri msg action load-config-file
```

验证新启动的进程能拿到变量：

```bash
rm -f /tmp/opencode/niri-env-check.txt
niri msg action spawn -- sh -c \
  'env | grep -E "^(XMODIFIERS|GTK_IM_MODULE|QT_IM_MODULE)=" > /tmp/opencode/niri-env-check.txt'
sleep 1
cat /tmp/opencode/niri-env-check.txt
# 期望：XMODIFIERS=@im=fcitx
```

说明：niri 的 `environment` 块在配置热加载后会对**之后启动的子进程**生效，
已经在运行的 Typora 不会自动获得新环境，需要重启 Typora。

### 10.4 修改二：给 Typora 单独加启动环境

新建用户级 desktop 覆盖文件 `~/.local/share/applications/typora.desktop`：

```ini
[Desktop Entry]
Name=Typora
GenericName=Markdown Editor
Exec=env XMODIFIERS=@im=fcitx GTK_IM_MODULE=fcitx QT_IM_MODULE=fcitx /usr/bin/typora %U
Icon=typora
Type=Application
StartupNotify=true
Categories=Office;WordProcessor;
MimeType=text/markdown;text/x-markdown;
```

这样无论从应用菜单、文件管理器双击 `.md`，还是通过 `xdg-open` 启动，
Typora 都会带上 XIM/GTK 的输入法环境变量。

校验：

```bash
desktop-file-validate ~/.local/share/applications/typora.desktop
```

### 10.5 验证 XIM 是否可用

```bash
XMODIFIERS=@im=fcitx fcitx5-diagnose 2>/dev/null | sed -n '/## Xim/,/## Qt/p'
```

期望输出：

```text
环境变量 XMODIFIERS 已经正确地设为了“@im=fcitx”。
从环境变量中获取的 Xim 服务名称为 fcitx.
根窗口上的 XIM_SERVERS：Xim 服务的名称与环境变量中设置的相同。
```

### 10.6 使改动生效

Typora 进程只在启动时读取环境变量，因此必须**完全退出 Typora 再重新打开**
（仅关闭窗口到托盘不算）。现有文档如有未保存内容，先保存。

### 10.7 回滚

- niri：从 `~/.config/niri/cfg/misc.kdl` 的 `environment` 块删除 `XMODIFIERS "@im=fcitx"`，
  然后 `niri msg action load-config-file`；
- Typora：删除 `~/.local/share/applications/typora.desktop` 即可恢复系统默认启动项。

# 最后

改完这些设置输入法，导入词库

[个人词库](../个人词库.txt)
