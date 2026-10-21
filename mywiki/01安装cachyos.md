# 跟着引导

###  分区


    这一步是重点，根据需求选择。
    
    - install alongside （并存）
    
        如果你是一个盘装 Win + Cachy 双系统，进 Live 环境前没有事先腾出硬盘空间，且 Win 的 EFI 分区大于 512MB，选择这一项。
    
    - replace a partition （替代一个分区）
    
        如果你是一个盘装 Win + Cachy 双系统，进 Live 环境前事先腾出了硬盘空间，且 Win 的 EFI 分区大于 512MB，选择这一项。
    
    - erase disk （抹掉磁盘）
    
        如果你使用一整个硬盘安装 Cachy，选择这一项。
    
    - manual partitioning（手动分区）
    
        如果以上都不满足，选这一项。

###  手动分区

    分出两个分区，一个启动分区，一个根分区。
    
    - 选择空闲空间，点击 create 创建。
    
    - 新建启动分区
    
        CachyOS 要求启动分区 Size（大小）至少为 `512MB`，FileSystem（文件系统）为 `fat32`，MountPoint（挂载点）为 `/boot/efi`，Flags（标记）为 `boot`。
    
    - 新建根分区
    
        剩下的空闲空间都是根分区，文件系统推荐 `btrfs`，挂载点为 `/`。

桌面选择niri

# 一行代码配置


```bash
curl -L shorin.xyz/archsetup | bash
```

桌面端选No_Desktop，防止影响cachy默认配置

- IWD Wifi Backend

  如果你安装了 NetworkManager，这一项会将 WiFi 后端从 `wpa_supplicant` 修改为 `iwd`。以获得更稳定、轻量、快速的 WiFi 体验。

- Windows Linux Dualboot Setup 双系统配置

  如果你的电脑上同时安装了 Windows 和 Linux，并且引导加载程序使用的是 GRUB，会自动帮你配置双系统的引导。

- Hardware Drivers 硬件驱动

  这一项会使用 `chwd-arch-git` 这个包安装上 `chwd`，运行 `chwd -a` 命令自动安装你的硬件需要的驱动。

- Grub Themes GRUB 主题美化

  如果你使用的是 GRUB，会出现一些可选的 GRUB 主题。

- Common Apps 常用软件

  启用这一步会在安装完桌面之后安装我推荐的常用软件，可以通过 TUI 列表挑选你要安装的。

  


# 常用软件

  这里是所有的常用软件，GUI 是图形化交互软件，TUI 是基于终端的交互软件。如果你有不需要的可以自己 `yay -Rns 包名` 删除。如果有残留的快捷方式可以查看 `~/.local/share/applications` 或者 `/usr/share/applications` 目录删除对应的 .desktop 文件。

  <details close><summary>常用软件列表</summary>
  

  - 互联网与社交
  
    | 软件包                      | 说明                         |
    | :-------------------------- | :--------------------------- |
    | `firefox` `python-pywalfox` | 需要    火狐浏览器和主题同步 |
    | `linuxqq-appimage`          | 需要    (AUR) QQ             |
    | `wechat-appimage`           | 需要    (AUR) 微信           |
    | `flclash-bin`               | 需要    (AUR) 网络代理工具   |
    | `localsend`                 | 需要    局域网传输神器       |
    | `nm-connection-editor`      | 高级网络配置管理             |
    | `transmission-gtk`          | 种子下载器                   |
    | `video-downloader`          | (AUR) 视频下载器 (B站/油管)  |
  
  - 游戏 (Gaming)
  
    | 软件包                      | 说明                            |
    | :-------------------------- | :------------------------------ |
    | `steam`                     | 需要    游戏平台                |
    | `lutris`                    | Wine 前缀和游戏库管理           |
    | `heroic-games-launcher-bin` | Epic/GOG 游戏管理               |
    | `protonplus`                | Proton 版本管理                 |
    | `mangohud`                  | 游戏性能监控浮层（Afterburner） |
    | `mangojuice-bin`            | (AUR) Mangohud 配置 GUI         |
    | `gamescope`                 | 游戏窗口合成器                  |
    | `lsfg-vk-bin`               | (AUR) 小黄鸭游戏补帧工具        |
    | `wine`                      | 运行 Windows 程序               |
  
  - 生产力与多媒体
  
    | 软件包                                           | 说明                              |
    | :----------------------------------------------- | :-------------------------------- |
    | `visual-studio-code-bin`                         | 需要    (AUR) VS Code 代码编辑器  |
    | `neovim`                                         | 需要    现代化的 Vim 终端编辑器   |
    | `lazyvim`                                        | 需要    优秀的 Neovim 预设配置    |
    | `opencode`                                       | 需要    开源 AI 助手              |
    | `mpv`                                            | 需要    视频播放器                |
    | `imv`                                            | 需要    图片查看工具              |
    | `obs-studio`                                     | 推流与录屏                        |
    | `upscaler`                                       | 图片无损放大 GUI                  |
    | `virt-manager` `qemu-full` `swtpm` `virt-viewer` | KVM 虚拟机管理                    |
    | `com.github.wwmm.easyeffects`                    | （flatpak）音频特效 (降噪/均衡器) |
  
  - 系统工具
  
    | 软件包                          | 说明                                  |
    | :------------------------------ | :------------------------------------ |
    | `nmtui`                         | TUI 网络连接配置工具                  |
    | `mission-center`                | 需要    系统监视器 (Win11 风格)       |
    | `btop`                          | 需要    TUI 系统监视器                |
    | `gdu`                           | 需要    TUI 磁盘占用分析工具          |
    | `baobab`                        | 需要    GUI 磁盘占用分析工具          |
    | `gparted`                       | 磁盘管理工具                          |
    | `gnome-font-viewer`             | 字体管理器                            |
    | `thunar`                        | GUI 文档管理器                        |
    | `yazi`                          | TUI 文档管理器                        |
    | `fcitx5-mozc`                   | 日语输入法                            |
    | `rime-wubi`                     | 五笔输入法                            |
    | `lact`                          | GPU 控制工具                          |
    | `pavucontrol`                   | 音频配置 GUI                          |
    | `flatseal`                      | Flatpak 权限管理                      |
    | `it.mijorus.gearlever`          | 需要       （flatpak）AppImage 管理器 |
    | `io.github.fabrialberio.pinapp` | （flatpak）.desktop 文件编辑工具      |
    | `bazaar`                        | 需要          flatpak 软件商城        |
    | `gnome-clocks` `gnome-calendar` | 时钟和日历                            |
    | `file-roller`                   | 需要           压缩解压缩             |

  </details>

#   总结

  - 互联网与社交

    | 软件包                      | 说明                         |
    | :-------------------------- | :--------------------------- |
    | `firefox` `python-pywalfox` | 需要    火狐浏览器和主题同步 |
    | `linuxqq-appimage`          | 需要    (AUR) QQ             |
    | `wechat-appimage`           | 需要    (AUR) 微信           |
    | `flclash-bin`               | 需要    (AUR) 网络代理工具   |
    | `localsend`                 | 需要    局域网传输神器       |

  - 生产力与多媒体

    | 软件包                   | 说明                             |
    | :----------------------- | :------------------------------- |
    | `visual-studio-code-bin` | 需要    (AUR) VS Code 代码编辑器 |
    | `neovim`                 | 需要    现代化的 Vim 终端编辑器  |
    | `lazyvim`                | 需要    优秀的 Neovim 预设配置   |
    | `opencode`               | 需要    开源 AI 助手             |
    | `mpv`                    | 需要    视频播放器               |
    | `imv`                    | 需要    图片查看工具             |

  - 系统工具

    | 软件包                 | 说明                                  |
    | :--------------------- | :------------------------------------ |
    | `steam`                | 需要    游戏平台                      |
    | `mission-center`       | 需要    系统监视器 (Win11 风格)       |
    | `btop`                 | 需要    TUI 系统监视器                |
    | `gdu`                  | 需要    TUI 磁盘占用分析工具          |
    | `baobab`               | 需要    GUI 磁盘占用分析工具          |
    | `it.mijorus.gearlever` | 需要       （flatpak）AppImage 管理器 |
    | `bazaar`               | 需要          flatpak 软件商城        |
    | `file-roller`          | 需要           压缩解压缩             |



# 完成之后

登录微信或qq 配置代理

在GitHub上下载niri配置，替换文件

配置过程在 [niri更改配置](../niri 键位配置修改说明.md)

| 网易云音乐 | yay -S splayer   |
| ---------- | :--------------- |
| 水杉输入法 | yay -S msime-bin |

用ai（opencode终端版）参考文档修改输入法

[删除rimesrf](../删除rime输入法.md)