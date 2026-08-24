# ENV 备忘
### 目录
- Vim
  - vim-plug
  - `.vimrc`（我就是喜欢用 Vim）
- Z Shell
- Linux
  - Linux配置Spice Agent
  - Linux桥接网卡配置
- Windows
  - Windows Powershell 编码
  - Windows 脚本执行策略更改
  - Windows Scoop 包管理器
  - Windows开启SSH隧道
  - Windows MSYS2 & gsudo & UAC & MinGW & Zsh
  - Unicode UTF-8
  - 关闭 Shift 切换输入法
  - 关闭 Defender
  - 安装 uutils/coreutils
- Wezterm
- VSCode
- 跨平台应用

# Vim — 我非常喜欢的编辑器
## vim-plug
> 一个 Vim 的依赖管理工具

*Windows*
```powershell
iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |`
    ni "$HOME/vimfiles/autoload/plug.vim" -Force
```

*Unix-like*
```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## `.vimrc`
``` Vim Script
" 插件
call plug#begin('~/.vim/plugged')
Plug 'catppuccin/vim', { 'as': 'catppuccin' }
call plug#end()

" 开启真彩支持（使用 Catppuccin 必须开启，否则会有色差）
if (has("termguicolors"))
    set termguicolors
endif

" 语法高亮
syntax on
" 行数显示
set number
" 缩进4格
set tabstop=4
" 自动缩进4格
set shiftwidth=4
" 缩进自动转换空格
set expandtab
" 自动缩进
set autoindent
" 鼠标支持
set mouse=a
" 抹茶
colorscheme catppuccin_mocha

" 切换透明/抹茶！背景
nnoremap <Leader>bg :call ToggleTransparent()<CR>

function! ToggleTransparent()
    if !exists('g:transparent_enabled')
        let g:transparent_enabled = 0
    endif
    if g:transparent_enabled == 0
        hi Normal ctermbg=NONE guibg=NONE
        let g:transparent_enabled = 1
    else
        " 重载主题配色
        colorscheme catppuccin_mocha
        let g:transparent_enabled = 0
    endif
endfunction

" 剪贴板
if has('clipboard')
    set clipboard=unnamedplus
    vnoremap <Leader>y "+y
    nnoremap <Leader>p "+p
    autocmd VimEnter * echom ":: clipboard enabled ::"
endif

" GUI 字体
if has('gui_running')
    " Linux     - set guifont=JetBrains\ Mono\ 11
    " Windows   - set guifont=JetBrains_Mono:h11
    " macOS     - set guifont=JetBrains\ Mono:h12

    if exists('+guifontwide')
        " Linux     - set guifontwide=Noto\ Sans\ Mono\ CJK\ SC\ 11
        " Windows   - set guifontwide=黑体:h11
        " macOS     - set guifontwide=Noto\ Sans\ Mono\ CJK\ SC:h12
    endif

    set lines=50
    set columns=100
endif

```

然后打开 Vim，输入 `:PlugInstall` 就行

# Z Shell
### 1. 安装 Oh My Zsh
```bash
# 安装 Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 安装 Powerlevel10k
```bash
# 安装 Powerlevel10k
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
```zsh
ZSH_THEME="powerlevel10k/powerlevel10k"
```
```bash
# 设置命令返回值为 0 时不输出 status
if [[ $(sed --version 2>&1) =~ "GNU" ]]; then
  sed -i '' -e 's/POWERLEVEL9K_STATUS_OK=true/POWERLEVEL9K_STATUS_OK=false/' -e 's/POWERLEVEL9K_STATUS_OK_PIPE=true/POWERLEVEL9K_STATUS_OK_PIPE=false/' ~/.p10k.zsh
else
  sed -i -e 's/POWERLEVEL9K_STATUS_OK=true/POWERLEVEL9K_STATUS_OK=false/' -e 's/POWERLEVEL9K_STATUS_OK_PIPE=true/POWERLEVEL9K_STATUS_OK_PIPE=false/' ~/.p10k.zsh
fi
```

# Linux — 是世界上最强的操作系统，没有之一

## Linux配置Spice Agent
```bash
# 从apt镜像源下载spice-vdagent
sudo apt install spice-vdagent

# 编辑自启动项
sudo vim /lib/systemd/system/spice-vdagent.service
```
---
> spice-vdagent.service 内容
```bash
[Unit]
Description=Agent daemon for Spice guests
Requires=spice-vdagentd.socket

[Service]
Type=forking
EnvironmentFile=-/etc/default/spice-vdagentd
ExecStart=/usr/sbin/spice-vdagentd $SPICE_VDAGENTD_EXTRA_ARGS
PIDFile=/run/spice-vdagentd/spice-vdagentd.pid
PrivateTmp=true
Restart=on-failure

[Install]
Also=spice-vdagentd.socket

# 添加下面的自启动内容
[Install]
WantedBy=multi-user.target
```
---
```bash
# 设置开机自启动
sudo systemctl enable spice-vdagent
```

## Linux桥接设置
### 方式
- 通用ip命令
- NetworkManager的nmcli命令
## IP命令
### 1. 查看物理网卡
```bash
# 1. 查看所有网络设备，找到物理网卡（通常带 "UP" 状态，连接物理网络）
sudo ip link show

# 2. 查看物理网卡的 IP 和子网掩码（示例：物理网卡为 eth0，IP 192.168.3.21/24，网关 192.168.3.1）
sudo ip addr show eth0  # 替换 eth0 为你的物理网卡名
sudo ip route show      # 查看默认网关（找 "default via" 那行）
```

### 2. 创建bridge设备
```bash
# 创建 bridge 设备 br0
sudo ip link add name br0 type bridge

# 启动 br0（相当于激活交换机）
sudo ip link set br0 up

# 验证 bridge 创建成功（应显示 br0 状态为 UP）
sudo ip link show br0
```

### 3. 将物理网卡加入bridge
```bash
# 1. 删除物理网卡的原有 IP（避免路由冲突）
sudo ip addr del phy_ip/24 dev phy_nic  # 替换：phy_ip=192.168.3.21，phy_nic=eth0

# 2. 将物理网卡绑定到 br0（相当于把网线插在交换机上）
sudo ip link set dev phy_nic master br0
```

### 4. 给bridge配置IP和网关
```bash
# 1. 给 br0 配置原物理网卡的 IP
sudo ip addr add phy_ip/24 dev br0  # 示例：sudo ip addr add 192.168.3.21/24 dev br0

# 2. 配置默认网关（确保宿主机能上外网，虚拟机也会继承此路由）
sudo ip route add default via gateway_ip dev br0  # 示例：sudo ip route add default via 192.168.3.1 dev br0

# 3. 验证 bridge 网络（ping 网关和外网，确保通）
sudo ping -c 2 gateway_ip  # 示例：ping -c 2 192.168.3.1
sudo ping -c 2 baidu.com   # 验证外网连通性
```

## nmcli命令
### 1. 创建桥接
```bash
# 创建br0桥接
nmcli connection add type bridge ifname br0 con-name br0

# 设置br0自动配置
nmcli connection modify br0 ipv4.method auto ipv6.method ignore
```

### 2. 将物理网口接入网桥
```bash
# 接入网桥
nmcli connection add type bridge-slave ifname enp3s0 master br0 con-name br0-slave

# 删除物理网口配置
nmcli connection delete enp3s0
```

### 3. 启用桥接
```bash
# 启用网桥
nmcli connection up br0

# 重启NetworkManager
systemctl restart NetworkManager
```
> 如果有VPN请关闭（需要刷新DNS）

# Windows — 难评，总之我不喜欢

## Windows Powershell编码
*1. 编辑 `$PROFILE`*
```powerhsell
# 设置 UTF-8
$OutputEncoding = [console]::InputEncoding = [console]::OutputEncoding = [Text.UTF8Encoding]::UTF8
```

*2. 检查 代码页*
```powershell
chcp
```

**注意！！**
> cat (**Get-Content**) 依旧使用其他编码，要加上 -Encoding utf8

## Windows 脚本执行策略更改
```powershell
# 更改
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
# 加载配置文件
. $PROFILE
```

## Windows 安装 Scoop 包管理器
```powershell
iwr -useb get.scoop.sh | iex
```

## Windows MSYS2 & MinGW
### MSYS2

*1. 安装*

```bash
scoop install msys2
```

*2. 完整命令*
```cmd
C:/Users/<username>/scoop/apps/msys2/current/msys2_shell.cmd -defterm -here -no-start -<env (e.g. ucrt64, clangarm64)> -shell <shell (e.g. bash, zsh)> -use-full-path
```

*3. 编辑配置*
```bash
# 将 db_home 的值改为 windows
vim scoop/app/msys2/current/etc/nsswitch.conf
```

## Windows 安装 gsudo
```bash
scoop install gsudo
```

## Windows 关闭 UAC

`Super + R` 输入 `UserAccountControlSettings` 调节并重启


### MinGW
```
# 对于大部分电脑
pacman -S mingw-w64-ucrt-x86_64-toolchain
```


## Windows配置SSH Server
### 1. 以管理员身份运行PowerShell
### 2. 安装并部署SSH Server
```powershell
# 下载SSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动并设置开机自启
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# 开放防火墙
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

# 查看状态（应为Running）
Get-Service sshd
```

## Windows 全局 Unicode UTF-8
`Win + R` 运行 `intl.cpl`，`管理` 选项卡中 `更改系统区域设置`，勾选 `Beta 版：使用 Unicode UTF-8 提供全球语言支持`


## 关闭 Shift 切换输入法！！

“设置” -> “时间与语言” -> “语言” --> “首选语言” --> 点击“中文（简体，中国）” --> 点击弹出的“选项” --> “键盘” --> 点击“微软拼音” --> 点击弹出的“选项” --> “按键” --> 关闭！

## 关闭 Defender（不赘述）

## 安装 uutils/coreutils

```bash
scoop install uutils-coreutils
export PATH="$HOME/scoop/shims:$PATH"
```

## WezTerm
### 安装 Nightly！

- Windows：建议在官网而不是 scoop
- 

```lua
local wezterm = require 'wezterm'

local config = wezterm.config_builder()

-- [ 1. 基础外观设置 ] --
config.color_scheme = 'Catppuccin Mocha'    -- 主题配色
config.font = wezterm.font_with_fallback({
  -- 1. 主字体（通常是英文等宽字体）
  { family = 'JetBrains Mono', weight = 'Regular' },
  
  -- 2. 第一回退字体（比如你喜欢的中文非等宽或等宽字体）
  { family = 'Noto Sans Mono CJK SC', weight = 'Regular' },
  
  -- 3. 图标回退字体（Nerd Fonts）
  { family = 'Noto Emoji', weight = 'Regular' },
})

config.font_size = 12                       -- 字体大小
config.use_fancy_tab_bar = false            -- 花稍标签栏
config.window_background_opacity = 1        -- 窗口不透明度

---- [ MSYS2 ] ----
--[[ config.default_prog = {
    'C:/Users/user/scoop/apps/msys2/current/msys2_shell.cmd',
    '-defterm', '-here', '-no-start',
    '-<env (e.g. ucrt64, clangarm64)>',
    '-shell', '<shell (e.g. bash, zsh)>', '-use-full-path'
}
--]]

-- [ 2. 背景图片与透明度设置 ] --

config.background = {
  {
    -- 冷灰色图层
    source = { Color = "#10161d" },
    width = "100%",
    height = "100%",
  },
  {
    source = { File = "<image>" },
    opacity = 0.15,                 -- 不透明度
    attachment = 'Fixed',           -- 混合叠加
    horizontal_align = 'Center',    -- 图片宽度居中
    hsb = {
        saturation = 0.85,          -- 饱和度
        brightness = 0.7            -- 亮度
    }
  },
}


---- [ 背景模糊 ] ----

---> Windows
-- 'Acrylic' (经典的亚克力高斯模糊，最推荐)
-- 'Mica' (Win11 特有的云母材质，跟随桌面壁纸色调，淡淡的模糊)
-- 'Tabbed' (Win11 标签页风格模糊)
config.win32_system_backdrop = 'Acrylic'

---> Linux
-- 开启 Linux 窗口半透明
config.window_background_opacity = 0.75
config.text_background_opacity = 1.0

---> macOS
config.window_background_opacity = 0.85
-- macOS 窗口背景模糊半径（数值越大越模糊）
config.macos_window_background_blur_radius = 30

-- [可选] 确保文本背景不透明，防止文字也跟着变模糊发虚
config.text_background_opacity = 1.0

---- [ Windows 无响应 ] ----
-- 1. 当关闭窗口时，不再为了等待这些进程结束而弹窗或卡死
config.skip_close_confirmation_for_processes_named = {
  'bash', 'sh', 'zsh', 'msys2', 'pacman', 'conhost.exe'
}

config.win32_system_backdrop = 'Disable'


return config
```

## VSCdoe 配置
打开 `Preferences: Open User Settings (JSON)` 编辑

### MSYS2 终端

```json
{
  ... 其他设置

    "terminal.integrated.profiles.windows": {
        // 自定义一个终端
        "MSYS2": {
            "path": "C:\\Users\\<username>\\scoop\\apps\\msys2\\current\\msys2_shell.cmd",
            "args": [
                "-defterm",
                "-here",
                "-no-start",
                "-<env>",
                "-shell",
                "<shell>",
                "-use-full-path"
            ]
        }
    },

    // 将其设置为 Windows 系统的默认终端
    "terminal.integrated.defaultProfile.windows": "MSYS2"
}
```

### 字体

```json
{
  "editor.fontSize": 14,
  "editor.fontWeight": "normal",
  "editor.fontFamily": "'JetBrains Mono', 'Noto Sans Mono CJK SC', monospace",
  "editor.fontLigatures": false   // 取消连字
}
```

### 在终端禁用编辑器快捷键
```json
{
    "terminal.integrated.sendKeybindingsToShell": true
}
```


## 跨平台应用

- 快速音频/图片/视频预览：*QuickLook (Windows)*，*Nyre221/Kiview (Linux Plasma)*
- 专门歌曲库管理/播放：*Kid3*，*Tauon Music Box*
- 专门图片管理/查看/编辑：*Nomacs*
- 专门视频播放：*mpv*

- DAW：*Ardour* & *LMMS*
- 复杂图像处理：*PhotoGIMP*，*Darktable*，*RapidRAW*
- 视频剪辑：*Kdenlive*，*DaVinci Resolve*
