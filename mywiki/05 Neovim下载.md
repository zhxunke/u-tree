CachyOS 基于 Arch,用 pacman 安装依赖即可。

### 1. 安装依赖

bash

```bash
sudo pacman -S --needed neovim git base-devel ripgrep fd lazygit fzf tree-sitter-cli nodejs npm
```

- `neovim`:LazyVim 要求较新的版本(目前需要 0.11 以上),Arch 仓库里的通常够新。
- `base-devel`:提供 gcc/make,用来编译 treesitter 解析器。
- `ripgrep`、`fd`、`lazygit`、`fzf`:搜索和 Git 功能用到。
- `nodejs npm`:部分 LSP 通过 Mason 安装时需要。

另外需要一个 Nerd Font,否则图标会显示成方块:

bash

```bash
sudo pacman -S ttf-jetbrains-mono-nerd
```

安装后在终端设置里把字体改成 JetBrainsMono Nerd Font。

### 2. 备份旧配置(如果有)

bash

```bash
mv ~/.config/nvim{,.bak}
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

### 3. 克隆 LazyVim starter

bash

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

### 4. 启动

bash

```bash
nvim
```

第一次启动会自动用 lazy.nvim 下载并安装所有插件,等它跑完后重启一次 nvim。然后运行 `:LazyHealth` 检查有没有缺失的依赖。

打开时出现问题

[neovim问题解决](../neovim-open-with-修复记录.md)