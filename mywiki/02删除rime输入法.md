你是用 pacman 装的包,所以按"包安装"处理,不用像我上一条那样手动删文件。

### 1. 先确认没有别的包依赖雾凇

bash

```bash
pacman -Qi rime-ice-git | grep -iE "required by|被.*需要"
```

如果水杉输入法(或别的包)依赖它,这里会列出来。显示 `None` 就可以放心删。

### 2. 卸载包

bash

```bash
sudo pacman -Rns rime-ice-git
```

`-s` 会连带删除不再被需要的依赖,执行前看清它列出的清单。如果你按教程装过语法模型包,也一并删:

bash

```bash
sudo pacman -Rns rime-wanxiang-gram-zh-hans   # 没装过会提示找不到,忽略即可
```

### 3. 清理用户目录里的残留

包删掉后,`~/.local/share/fcitx5/rime` 里你自己写的配置还在,而且还引用着雾凇。这是最需要处理的一步,不然 Rime 部署时会报找不到 `rime_ice_suggestion`。

bash

```bash
cd ~/.local/share/fcitx5/rime
ls
```

按你的教程,可能有这些文件:

- `default.custom.yaml`:删掉 `__include: rime_ice_suggestion:/` 这一行。如果里面的 `schema_list` 有 `- schema: rime_ice`,也一并删掉。
- `rime_ice.custom.yaml`:这是语法模型和英文标点的配置,直接删除。
- `wanxiang-lts-zh-hans.gram`:如果是手动下载放进来的,删除。
- `rime_ice.userdb/`:雾凇的个人词频,不想留就删(会丢失词频学习记录)。
- `build/`:编译缓存,删掉让它重建。
- `custom_phrase.txt`:这是你自己的自定义词库,如果水杉也走 Rime 就保留,否则自己决定。

bash

```bash
rm -f rime_ice.custom.yaml wanxiang-lts-zh-hans.gram
rm -rf rime_ice.userdb build
```

### 4. 重新部署

bash

```bash
fcitx5 -r
```

然后打开 `fcitx5-configtool`,确认输入法列表里没有多余的 Rime 条目,只留水杉。

### 5. 关于 fcitx5-rime 本身

如果水杉**不是**基于 Rime 的,你可以把 Rime 整个卸掉:

bash

```bash
sudo pacman -Rns fcitx5-rime
rm -rf ~/.local/share/fcitx5/rime
```

如果水杉是基于 Rime 的方案,就保留 `fcitx5-rime`,只在 `default.custom.yaml` 里把方案改成水杉的方案 ID。

环境变量和 niri 里的 `spawn-at-startup "fcitx5"` 都不用动,因为框架还是 fcitx5。

我对水杉输入法的具体实现不确定,不知道它是不是走 Rime。你可以告诉我它是怎么装的(包名或安装方式),我再帮你确认第 5 步该怎么选。

接下来是对水杉的调整

[水杉输入法配置](../水杉输入法-fcitx5默认中文配置记录.md)

在完全配置好之后，重启一次，再进行输入法账号登录与设置。

