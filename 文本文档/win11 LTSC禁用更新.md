Windows 11 **24H2** 的设置界面如果是英文（Settings app 显示为 English），组策略编辑器（gpedit.msc）里的路径和策略名称也是英文的。以下是针对 **Windows 11 Enterprise LTSC**（或普通企业版）在英文界面下**禁止/极度限制更新**的最实用步骤（和之前中文版对应，但用英文路径）：

### 步骤：打开组策略编辑器
1. 按 **Win + R** 键，输入 `gpedit.msc` 并按 Enter 打开 **Local Group Policy Editor**。

### 推荐组合（最强力禁止自动更新 + 断开微软服务器）
#### 1. 彻底禁用自动更新（最核心一步）
路径：  
**Computer Configuration** → **Administrative Templates** → **Windows Components** → **Windows Update** → **Manage end user experience**

- 双击 **Configure Automatic Updates**  
  - 选择 **Disabled**（已禁用）  
  - 点击 **Apply** → **OK**  
  （这一步让自动下载/安装全部失效，Windows Update 基本不动）

- （可选但强烈推荐）双击 **Remove access to use all Windows Update features**  
  - 选择 **Enabled**（已启用）  
  - Apply → OK  
  （这一步会让 Settings → Windows Update 页面显示“Some settings are hidden or managed by your organization”，检查更新按钮也无效或灰掉）

#### 2. 让系统连不到微软更新服务器（最狠，永久断开）
路径：  
**Computer Configuration** → **Administrative Templates** → **Windows Components** → **Windows Update**

- 双击 **Specify intranet Microsoft update service location**  
  - 选择 **Enabled**  
  - 在两个框（Set the intranet update service for detecting updates / Set the intranet statistics server）都填一个无效地址，例如：  
    http://127.0.0.1:9999  
  - Apply → OK  
  （系统就找不到更新服务器了，除非你自己有 WSUS）

#### 3. 额外禁止驱动自动更新（推荐，防止显卡/网卡驱动被乱改）
还是在 **Windows Update** 路径下：

- 双击 **Do not include drivers with Windows Updates**  
  - 选择 **Enabled**  
  - Apply → OK

### 快速生效
- 按 Win + R，输入 `cmd` 以管理员身份运行命令提示符  
- 输入以下命令并回车：  
  `gpupdate /force`  
- 然后重启电脑。

### 英文界面下的常见路径总结表（方便对照）

| 中文路径对应 | 英文完整路径 | 推荐设置 | 作用 |
|--------------|--------------|----------|------|
| 配置自动更新 | Configure Automatic Updates | Disabled | 关闭自动更新机制 |
| 删除使用所有 Windows 更新功能的访问权限 | Remove access to use all Windows Update features | Enabled | 隐藏/禁用更新页面大部分功能 |
| 指定 Intranet Microsoft 更新服务位置 | Specify intranet Microsoft update service location | Enabled + 填无效地址 | 断开微软服务器连接 |
| 不包括驱动程序与 Windows 更新 | Do not include drivers with Windows Updates | Enabled | 只禁驱动更新 |

做完这些后：
- Settings → Windows Update 通常会显示组织策略管理，或直接无法检查更新。
- LTSC 本身功能更新（如 24H2 → 25H2）基本不会来，质量/安全补丁也会被挡住。
- 如果你只是想**延迟**而不是完全禁止，可以在 **Windows Update for Business** 子路径下设置最大延迟天数（Feature updates deferral up to 365 days, Quality updates up to 30 days）。

**注意**：完全禁止更新有安全风险（尤其联网电脑），建议每 3–6 个月临时把这些策略改回 Not Configured / Disabled，打完补丁再关掉。或者用 WSUS 只放行你信任的 KB。

需要对应的注册表 .reg 文件（英文系统通用）或检查当前策略是否生效的命令吗？