

# 安装

```bash
DankMS脚本安装
curl -fsSL https://install.danklinux.com | sh
按提示选择，输入root密码

shorin配置脚本
curl -L shorin.xyz/archsetup | bash
按需跳过步骤


sudo pacman -S 
mission-center  类似win11的任务管理器
gnome-font-viewer 方便安装和查看字体
foliate 电子书阅读器
papers pdf阅读器
firefox浏览器
haruna是基于mpv的视频播放器
steam

paru -S linuxqq-appimage wechat-appimage
cnmplayer-git   TUI网易云播放器
clash-verge  好用
wps-office-cn wps办公
wps-office-mui-zh-cn wps的中文语言包
typora-free typora的免费版，markdown编辑器

paru -S watt-toolkit-bin    steam++
	hosts文件提权  sudo chmod a+w /etc/hosts
	保证端口无占用，无其他代理
```



# 锁屏

安装 greetd 和 DMS Greeter：

```bash
paru -S greetd greetd-dms-greeter-git

以下命令会关闭桌面，重启进tty输入niri重进桌面
# 禁用 SDDM
sudo systemctl disable --now sddm.service
# 启用 greetd
sudo systemctl enable --now greetd.service
# 配置 DMS Greeter（推荐用 dms 命令）
dms greeter enable
dms greeter sync
重启reboot
```

# 个性化

## 壁纸

外部壁纸管理  关 禁用

模糊壁纸层

## 主题与配色

光标主题 breeze_cursors

光标尺寸38px

## 排版与动画

字体缩放107%

动画速度 短

## 时间与天气

日期格式：日 月份 日期

# Dank Bar

## 	设置

​		边缘间距5%
​		独占区域偏移 -2%
​		尺寸10%
​		内边距 10px
​		字体缩放120%

# 工作区与部件

## 工作区

显示工作区内应用   上限  3

图标大小 2px

分区工作区应用   仅显示占用工作区  Drag to Reorder

## 程序坞与启动器

启动器按钮Logo    系统Logo   

尺寸偏移   5%























































