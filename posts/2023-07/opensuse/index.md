# openSUSE 安装和窗口管理器安装


## 安装

镜像下载地址：`https://get.opensuse.org/tumbleweed/#download`





## 窗口管理器安装

### 创建一个普通用户

```bash
#新建用户名arch 可自行更改用户名
useradd -m -G wheel -s /bin/bash arch

#设置arch用户名的密码
passwd arch

#编辑arch用户的权限
EDITOR=vim visudo

找到 # %wheel ALL=(ALL:ALL)ALL 并把 # 号去掉
```

**个人是注释掉 `%wheel ALL=(ALL:ALL)ALL` 这一行，如果不想使用 `sudo` 命令时输入密码，可以注释掉 `%wheel ALL=(ALL:ALL) NOPASSWD: ALL`。两个选择一个注释掉既可以了**



### 安装 dwm

重启使用普通用户登录

```bash
# 安装 git
sudo zypper install git

# 创建 wm 并进入文件夹
mkdir wm && cd ~/wm

# 克隆 dwm
git clone https://github.com/roukaixin/yaocccc-dwm

# 修改名字
mv yaocccc-dwm/ dwm 
cd dwm

# 安装编译包
sudo zypper install cmake gcc libX11-devel libXft-devel libXinerama-devel

# 编译
sudo make clean install
```



### 安装 st

```bash
# 克隆
cd ~/wm && git clone https://gitee.com/rouxin/st && cd st

# 修改 config.mk
sudo zypper install neovim ncurses-devel
nvim config.mk
# 取消注释，并把 c99 改为 gcc

# 编译
sudo make clean install
```

![image-20230715175157860](./note%20picture/openSUSE.assets/image-20230715175157860.png  " ")



### 安装 picom

```bash
# 克隆
cd ~/wm && git clone https://github.com/FT-Labs/picom 

# 安装包
sudo zypper install meson libev-devel xcb-util-renderutil-devel xcb-util-image-devel libpixman-1-0-devel xcb-util-devel uthash-devel libconfig-devel pcre2-devel Mesa-libGL-devel Mesa-libEGL-devel dbus-1-devel

# 编译安装
cd picom
git submodule update --init --recursive
meson setup --buildtype=release . build
ninja -C build install
```


### 安装字体

下载地址：`https://github.com/ryanoasis/nerd-fonts/releases/`

![image-20230714175135561](./note%20picture/openSUSE.assets/image-20230714175135561.png " ")

下载需要的字体，本人使用 dwm 的字体为：`JetBrainsMono.zip`，下载地址为：`https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/JetBrainsMono.zip`

```bash
# 创建目录
sudo mkdir /usr/share/fonts/JetBrainsMono/

# 复制文件到 /usr/share/fonts/JetBrainsMono/ 目录下
cp JetBrainsMono.tar.xz /usr/share/fonts/JetBrainsMono/

# 解压，如果没有 unzip 命令，使用 sudo dnf install unzip 安装
cd /usr/share/fonts/JetBrainsMono/ && sudo unzip JetBrainsMono.zip

# 删除
sudo rm -rf JetBrainsMono.zip
```


### 安装 xorg 服务器

```bash
sudo zypper install xorg-x11-server xinit

# 创建配置文件
nvim .xinitrc

#!/bin/bash
exec dwm
```



### 其他依赖包

```bash
sudo zypper install NetworkManager-applet flameshot dunst feh neofetch acpi xsetroot rofi
```



## 配置

### 启动中文

```bash
# 在 ～/.xintrx 中加入
# 注意 ： 一定要加在 exec dwm 前
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```



### 安装输入法

```bash
sudo zypper install fcitx5-devel fcitx5-chinese-addons-devel
```

> 配置输入法

```bash
# 编辑 /etc/environment，加入下面内容
vim /etc/environment

# 内容
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```



### 显示亮度

```bash
sudo zypper install brightnessctl
```



### 声卡驱动

```bash
sudo zypper install pipewire-pulseaudio pavucontrol
```



### 显卡驱动

```bash

```

---

> 作者:   
> URL: https://roukaixin.github.io/posts/2023-07/opensuse/  

