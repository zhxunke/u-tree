# KDE



## 桌面面板（任务栏）

​    右键任务栏（kde里叫面板），显示面板配置。设置为半透明；对齐，居中；宽度，适合内容；悬浮改成仅小程序；显示隐藏改成避开窗口；
​    删除工作区、显示桌面相关组件；添加两个间隔，把开始菜单和软件移动到中心。
​    更换面板图标，显示面板配置，应用程序启动器配置，换图标
右下角组件
​    点击时间左边的上箭头，在弹出来的窗口的右上角开启系统托盘设置，项目里面按需设置。设置电量和电池总是显示，蓝牙总是隐藏。
日历
​    配置添加农历与中国节日

## 系统设置里没有登录界面

​    sudo pacman -S sddm-kcm
​    重启系统设置
​    点击微风主题右下角图片，清楚，加载文件 自定义图片

## 设置

缩放显示和监视器，配置，缩放120%
颜色和主题，都用微风深色，欢迎屏幕，无；登录，微风；光标，30号
夜间颜色，总是适用夜间色温，色温4400K
字体，思源黑体cn，16号

## 桌面会话

​    启动为空会话
​    系统设置>搜索会话，应用

## 快捷键

系统设置>键盘>快捷键

- ​    终端  meta+F

- ​    KRunner   ctrl+ 空格

-    关闭窗口  alt+ Esc

- 添加程序，任务中心（missioncenter )  ctrl+shift+Esc

  设置>输入法>配置全局选项>切换输入法，左shift

截图软件设置
    打开spectacle的设置
    常规页面勾选“保存文件到默认文件夹”，再点击这行字下面的框，选择复制到剪贴板。

## sddm寄了

​    ctrl+alt+F4 进tty
​    补全依赖

```bash
   sudo pacman -Syu sddm qt5-graphicaleffects qt5-quickcontrols2 qt5-svg qt6-base qt6-declarative
```

​    重启sddm

```bash
  sudo systemctl restart sddm
```

## wallpaper

KDE插件（安装慢要等几分钟）

```bash
paru -S wallpaper-engine-kde-plugin
```

[github地址](https://github.com/catsout/wallpaper-engine-kde-plugin)

另外需要python包

```bash
sudo pacman -S python-websockets
```

# 安装

```bash
yay -S clash-verge
paru -S typora-free
paru -S localsend
图片编辑查看工具
pacman -S gwenview
网易云播放器
yay -S splayer-bin
音频条
pacman -S cava
```



## QQ

​    yay -S linuxqq-appimage

​	无法找到 fakeroot 的二进制文件。
​    运行下面这条命令安装完整的开发工具组：
​    sudo pacman -S --needed base-devel

### qq输入没反应

​    确认用的是 Wayland 还是 X11
​    echo $XDG_SESSION_TYPE
​    Wayland
​    在 ~/.config/gtk-3.0/settings.ini（没有就新建）里加：
​    [Settings]
​    gtk-im-module=fcitx

## 安装同花顺

​    yay -S cn.com.10jqka
​    sudo pacman -S openssl-1.1  # 老版本依赖 openssl1.1

## WPS与其字体

paru -S wps-office-cn
paru -S wps-office-mui-zh-cn

## firefox

隐私声明拉到最底下改中文
https://www.mozilla.org/zh-CN/privacy/firefox/

# 终端

```bash
sudo pacman -S fish
改为默认shell
chsh -s /usr/bin/fish
编辑配置文件去掉默认的启动文字
vim ~/.config/fish/config.fish
set fish_greeting ""
##     
```

## 提示符

​    sudo pacman -S ttf-jetbrains-mono-nerd starship
​    启用提示符
​    vim ~/.config/fish/config.fish
​    starship init fish | source
​    提示符主题
​    starship preset pastel-powerline -o ~/.config/starship.toml

## Konsole美化

1. 菜单>设置>显示工具栏>去掉两个勾选

2. 右键>菜单>设置>配置konsole

   常规页面里激活“移除窗口标题和框架”。

   配置方案里新建一个配置方案；外观里点击“获取新方案：下载一个自己喜欢的，我使用catppuccin  frappe。选中喜欢的配色方案后点击编辑设置20%透明度；设置字体为Adwaita  Mono（一个你喜欢的mono等宽字体），大小15pt；”其他“页面里设置边距，取消激活调整大小后显示终端大小提示。

   滚动里隐藏滚动条，取消激活高亮显示刚刚进入视图的行。确认。

   选中刚刚创建的配置方案设置为默认。

   确认。

   重启终端。

## vim

```bash
sudo pacman -S gvim
安装过程中会提示移除 vim 包（这是正常行为），确认即可。安装后 vim 命令仍然可用，且剪切板功能会启用
vim --version | grep -E 'clipboard|xterm_clipboard'
应该看到 +clipboard 和 +xterm_clipboard
```



# U盘手动挂载

查看分区格式 lsblk -pf

sudo pacman -S ntfs-3g gvfs gvfs-mtp udisks2

挂载

sudo mount -t ntfs-3g /dev/sda1 /mnt

# wine的使用

安装
sudo pacman -Syu wine wine-mono wine-gecko winetricks
字体
winetricks cjkfonts

## 想玩游戏先安装lutris

sudo pacman -S lutris wine wine-mono wine-gecko winetricks
AMD vulkan驱动
sudo pacman -S mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon
打开lutris 右上角，首选项，

# 输入法

​    mkdir -p ~/.local/share/fcitx5/rime
​    vim ~/.local/share/fcitx5/rime/default.custom.yaml
​    第一行命令mkdir -p检查文件夹是否存在，不存在的话创建。
​    在文件中写入
​    patch:
​    # 这里的 rime_ice_suggestion 为雾凇方案的默认预设
​      __include: rime_ice_suggestion:/
​      menu:
​        page_size: 9

```bash
右键输入法图标，放在雾凇拼音上，点重新部署

忽略坏字符
iconv -f gbk -t utf-8 -c rime_ice_export.txt -o fixed.txt
👉 -c 的作用：跳过非法字节保留能解析的内容
然后导入
rime_dict_manager --import rime_ice fixed.txt
```

