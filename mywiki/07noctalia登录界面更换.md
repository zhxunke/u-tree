# Noctalia Greeter 替换 SDDM：操作总结

适用环境：CachyOS + niri（Wayland）。方案是 SDDM → greetd + noctalia-greeter。

## 1. 确认安装

```bash
pacman -Q noctalia-greeter greetd
which noctalia-greeter-session
getent passwd greeter greetd
```

- `greetd` 没装就先 `sudo pacman -S greetd`，D-Bus 和 polkit 一般已有。
- `which` 的结果是后面 `command` 要填的路径，包安装一般是 `/usr/bin/noctalia-greeter-session`。
- `getent` 用来确认系统里实际存在的是 `greeter` 还是 `greetd` 用户。

## 2. 运行系统设置脚本（root）

脚本会创建 `/var/lib/noctalia-greeter/` 和 `greeter.conf`，并设好 greetd 用户的权限。

```bash
pacman -Ql noctalia-greeter | grep -i setup
```

找到脚本后用 root 运行。它还会打印一段现成的 `config.toml` 配置，直接用它的最稳。

## 3. 修改 greetd 配置

默认的 `/etc/greetd/config.toml` 启动的是 `agreety`（文本登录）：

```toml
[default_session]
command = "agreety --cmd /bin/sh"
user = "greeter"
```

改成：

```toml
[default_session]
command = "/usr/bin/noctalia-greeter-session"
user = "greeter"
```

- `command` 必须是 `noctalia-greeter-session` 的完整路径，不能写 `noctalia-greeter` 本体。
- `user` 要和第 1 步 `getent` 查到的用户一致。
- 旧的 `agreety` 那行要删掉或注释掉，别留两个 `command`。
- 想默认选中 niri：`command = "/usr/bin/noctalia-greeter-session -- --session niri"`，会话名用 `noctalia-greeter sessions` 查。
- 想跳过用户列表：在命令后加 `--user 你的用户名`。

编辑需要 root，例如 `sudoedit /etc/greetd/config.toml`。

## 4. 检查日志目录

黑屏排查依赖这两个日志，重启前确认存在：

```bash
ls -l /var/log/noctalia-greeter.log /var/lib/noctalia-greeter/
```

缺失时，需要手动创建并让 greetd 用户做所有者。

## 5. 切换显示管理器

```bash
sudo systemctl disable sddm
sudo systemctl enable greetd
```

确认前面几步无误后再重启。

## 6. 可选：同步壁纸和配色

需要 Noctalia v5。进入 设置 → Security → Noctalia Greeter → Sync Now，同步后注销或 `sudo systemctl restart greetd` 生效。同步需要 `pkexec` 或 `run0`，并且要有 polkit 认证代理在运行。

## 7. 出问题时

| 现象 | 原因 / 处理 |
|---|---|
| 黑屏 | 看 `/var/log/noctalia-greeter.log` 和 `/var/lib/noctalia-greeter/greeter.log` |
| `Failed to spawn client` | `command` 路径写错，用 `which` 的结果 |
| `WAYLAND_DISPLAY is not set` | `command` 用成了 `noctalia-greeter`，应为 `noctalia-greeter-session` |
| 启动后选错会话 | `greeter.conf` 里的 `default_session` 优先于上次选择 |
| 分辨率、缩放不对 | 在 `greeter.conf` 里设 `scale` |
| 光标主题不对 | 在 `greeter.conf` 里设 `cursor_theme` 和 `cursor_size` |

## 退路

按 `Ctrl+Alt+F3` 进 TTY，用 `journalctl -u greetd -b` 看日志；要回退到 SDDM：

```bash
sudo systemctl disable greetd
sudo systemctl enable sddm
```

## 登录界面快捷键

| 按键 | 作用 |
|---|---|
| `Tab` / `Shift+Tab` | 切换焦点 |
| `Enter` | 提交密码 / 确认 |
| `Esc` | 关闭菜单 / 退出密码步骤 |
| `F3` | 选择会话 |
| `F7` | 选择配色 |
| `Ctrl+Alt+F1–F12` | 切换到 TTY |

## 参考

- 文档：<https://docs.noctalia.dev/v5/greeter/>
- 仓库：<https://github.com/noctalia-dev/noctalia-greeter>
