# greetd 登录后钥匙环重复要密码

**环境**：CachyOS / niri / greetd 0.10.3 + Noctalia Greeter 1.7.0（原 SDDM）/ gnome-keyring 51.1
**现象**：greeter 里登录成功后，桌面又弹一次密码框（gcr-prompter，解锁登录密钥环），每次登录都这样。
**状态**：方案已确认，尚未写入系统（需要 root）；改完对下一次登录生效。

## 原因

gnome-keyring 的自动解锁依赖 PAM：`pam_gnome_keyring.so` 在 **auth** 阶段暂存登录密码，在同服务的 **session** 阶段以 `auto_start` 启动 `gnome-keyring-daemon` 并用它解锁 `~/.local/share/keyrings/login.keyring`。

SDDM 的 `/etc/pam.d/sddm` 自带这两行，所以以前不用再输；greetd 用的 `/etc/pam.d/greetd` 只有纯系统认证，没有钥匙环模块：

```pam
auth       required     pam_securetty.so
auth       requisite    pam_nologin.so
auth       include      system-local-login
account    include      system-local-login
session    include      system-local-login
```

`system-local-login → system-login → system-auth` 里也没有（`grep -rn pam_gnome_keyring /etc/pam.d/` 只有 sddm 相关文件命中）。于是密码验证成功但没交给钥匙环，钥匙环保持上锁，桌面 autostart 拉起 daemon 后弹框要密码。

这是 greetd 上游 PAM 文件的原始状态，与 Noctalia Greeter、niri 无关。

**证据**（本次开机）：

```console
20:10:33 greetd[917]: pam_unix(greetd:session): session opened for user xunke
20:10:34 systemd[963]: Started GNOME Keyring daemon.
20:10:34 gcr-prompter[1187]: Gcr: starting password prompt for callback /org/gnome/keyring/Prompt/p2@:1.27
20:10:40 gcr-prompter[1187]: Gcr: completed password prompt
```

登录 1 秒后 daemon 启动、随即弹框，6 秒后手输完成——就是那第二次输入。

## 修改

编辑 `/etc/pam.d/greetd`，在两条 `include` 之后各加一行：

```diff
 auth       include      system-local-login
+-auth      optional     pam_gnome_keyring.so
+
 account    include      system-local-login
+
 session    include      system-local-login
+-session   optional     pam_gnome_keyring.so    auto_start
```

或者直接执行：

```bash
sudo cp -a /etc/pam.d/greetd{,.bak}
sudo sed -i '/^auth .*include .*system-local-login/a -auth      optional     pam_gnome_keyring.so' /etc/pam.d/greetd
sudo sed -i '/^session .*include .*system-local-login/a -session   optional     pam_gnome_keyring.so    auto_start' /etc/pam.d/greetd
```

要点：

- **必须在 `include` 之后**：auth 行要等 `pam_unix` 校验完、密码进了 PAM handle 才能拿到；顺序与 SDDM 一致。
- **`optional` + `-` 前缀**：模块缺失或失败不影响登录，不会锁死系统。
- 不需要 `password` 行（greetd 不改密码，`passwd` 走自己的 PAM 文件）。
- greeter 会话共用这个 PAM 服务；greeter 无密码认证，模块直接放行，无副作用。

## 验证

注销重登（**不要** `systemctl restart greetd`，会杀掉当前会话），然后：

```bash
journalctl -b | grep -i gcr-prompter
```

不应再出现新的 `starting password prompt`（本次开机修复前的旧记录仍在，注意时间戳）。

## 回滚与维护

回滚：`sudo cp -a /etc/pam.d/greetd.bak /etc/pam.d/greetd`

- greetd 升级后本文件会生成 `.pacnew`，改动可能被重置；症状复现时先看 `journalctl -b | grep -i gcr-prompter`，再检查这两行是否还在。
- 本机没装 kwallet；若以后用 kwallet，需另加 `pam_kwallet5.so` 的 auth/session 行（可参考 sddm 配置）。
- 若将来改用 greetd 的 `initial_session` 自动登录，auth 栈会被完全跳过，此方案失效，需改用 `pam_fde_boot_pw`（见 Arch Wiki: Greetd）。
