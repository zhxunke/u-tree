# greetd 登录后钥匙环重复要密码

- 日期：2026-10-22 发现，2026-10-23 00:04 修复
- 环境：CachyOS / niri / greetd 0.10.3 + Noctalia Greeter 1.7.0（原 SDDM）/ gnome-keyring 51.1
- 现象：greeter 里登录成功后，桌面又弹一次密码框（gcr-prompter，解锁登录密钥环），每次登录都这样
- **最终状态：已根治。** 登录时 `pam_gnome_keyring` 直接把密码交给钥匙环，`gkr-pam: unlocked login keyring`，不再弹窗

## 一、原因

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

**修复前证据**（2026-10-22 20:10 开机）：

```console
20:10:33 greetd[917]: pam_unix(greetd:session): session opened for user xunke
20:10:34 systemd[963]: Started GNOME Keyring daemon.
20:10:34 gcr-prompter[1187]: Gcr: starting password prompt for callback /org/gnome/keyring/Prompt/p2@:1.27
20:10:40 gcr-prompter[1187]: Gcr: completed password prompt
```

登录 1 秒后 daemon 启动、随即弹框，6 秒后手输完成——就是那第二次输入。

## 二、修改内容

只改一个文件：`/etc/pam.d/greetd`，在两条 `include` 之后各加一行。

```diff
 auth       include      system-local-login
+-auth      optional     pam_gnome_keyring.so
+
 account    include      system-local-login
+
 session    include      system-local-login
+-session   optional     pam_gnome_keyring.so    auto_start
```

应用命令：

```bash
sudo cp -a /etc/pam.d/greetd /etc/pam.d/greetd.bak.$(date +%F)
sudo sed -i '/^auth .*include .*system-local-login/a -auth      optional     pam_gnome_keyring.so' /etc/pam.d/greetd
sudo sed -i '/^session .*include .*system-local-login/a -session   optional     pam_gnome_keyring.so    auto_start' /etc/pam.d/greetd
```

要点：

- **必须在 `include` 之后**：auth 行要等 `pam_unix` 校验完、密码进了 PAM handle 才能拿到；顺序与 SDDM 一致。
- **`optional` + `-` 前缀**：模块缺失或失败不影响登录，不会锁死系统。
- 不需要 `password` 行（greetd 不改密码，`passwd` 走自己的 PAM 文件）。
- greeter 会话共用这个 PAM 服务；greeter 无密码认证，模块直接放行，无副作用。

## 三、应用记录（2026-10-23 00:04）

已写入系统。应用后的文件内容（`cat -n /etc/pam.d/greetd`）：

```pam
 1  #%PAM-1.0
 2
 3  auth       required     pam_securetty.so
 4  auth       requisite    pam_nologin.so
 5  auth       include      system-local-login
 6  -auth      optional     pam_gnome_keyring.so
 7  account    include      system-local-login
 8  session    include      system-local-login
 9  -session   optional     pam_gnome_keyring.so    auto_start
```

- 第 6、9 行行尾各多一个空格，PAM 解析时忽略，无影响。
- 这次没有留下 `.bak`，回滚方式见第六节。
- 文件时间戳 2026-10-23 00:04，大小 220 → 326 字节。

**生效证据**（00:04:52 重新登录）：

```console
00:04:49 greetd[4891]: gkr-pam: couldn't unlock the login keyring.     ← greeter 用户，正常
00:04:52 greetd[5012]: gkr-pam: unlocked login keyring                 ← 目标用户，成功
00:04:52 greetd[5012]: pam_unix(greetd:session): session opened for user xunke
```

登录后 `pgrep -a gcr-prompter` 为空，`journalctl -b | grep 'gcr-prompter.*password prompt'` 在 00:04:52 之后没有任何新记录。

> greeter 用户那条 `couldn't unlock` 以及 `unable to create keyring dir: /.local/share/keyrings` 是 greeter 自己的会话产生的（greeter 无密码、没有自己的钥匙环），不影响用户会话，可忽略。

## 四、验证

注销重登（**不要** `systemctl restart greetd`，会杀掉当前会话），然后：

```bash
# 1. 两行还在，位置正确
grep -n pam_gnome_keyring /etc/pam.d/greetd

# 2. 应出现 unlocked login keyring，且没有新的 password prompt
journalctl -b | grep -iE 'gkr-pam|gcr-prompter.*password prompt'
```

正常输出形如：

```console
greetd[...]: gkr-pam: unlocked login keyring
```

修复前的旧 `gcr-prompter` 记录仍在 journal 里，按时间戳区分即可。

## 五、如果以后又弹了

按顺序查：

1. **确认改动还在**：`grep -c pam_gnome_keyring /etc/pam.d/greetd` 应为 2。为 0 就是被 greetd 升级重置了（见第六节）。

2. **看 PAM 有没有真的调用模块**：

   ```bash
   journalctl -b | grep -i 'gkr-pam'
   ```

   - 有 `unlocked login keyring` → 已解锁，弹窗来自别的东西（个别应用自己的密码框）。
   - 有 `couldn't unlock the login keyring` → 见下一条。
   - 完全没有 `gkr-pam` 行 → 配置没生效，检查文件语法、是否改错了文件（greetd 用的是 `/etc/pam.d/greetd`）。

3. **`couldn't unlock` = 钥匙环密码 ≠ 登录密码**：PAM 拿到登录密码但解不开钥匙环，此时弹窗里必须输旧密码才能通过。

   - 修复：装 `seahorse`（密码和密钥）→「登录」钥匙环 → 更改密码 → 改成当前登录密码。
   - 或者删掉 `~/.local/share/keyrings/login.keyring` 重建（会自动使用当前登录密码，代价是里面存的密钥全部丢失）。

## 六、回滚与维护

本次改动只有那两行，回滚就是删掉它们：

```bash
sudo sed -i '/pam_gnome_keyring/d' /etc/pam.d/greetd
```

若当初按第二节建了 `.bak.*`，也可以 `sudo cp -a /etc/pam.d/greetd.bak.* /etc/pam.d/greetd`。

- greetd 升级后本文件会生成 `.pacnew`，改动可能被重置；症状复现时先看 `journalctl -b | grep -i gcr-prompter`，再检查这两行是否还在。
- 本机没装 kwallet；若以后用 kwallet，需另加 `pam_kwallet5.so` 的 auth/session 行（可参考 sddm 配置）。
- 若将来改用 greetd 的 `initial_session` 自动登录，auth 栈会被完全跳过，此方案失效，需改用 `pam_fde_boot_pw`（见 Arch Wiki: Greetd）。
