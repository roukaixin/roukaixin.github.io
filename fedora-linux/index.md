# fedora Linux 安装








# 安装

> 下载`iso` 镜像

下载地址：`https://mirrors.tuna.tsinghua.edu.cn/`

下载带有 `Everything` 的镜像文件，这个镜像文件可以自定义安装。

![image-20230714150218400](./note%20picture/fedoraLinux.assets/image-20230714150218400.png)



> 安装

创建启动u盘，可视化安装教程。这里我安装的是最小化安装，因为我使用的是dwm窗口管理器





# 安装配置

## 替换为国内源

```bash
sudo sed -e 's|^metalink=|#metalink=|g' \
         -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
         -i.bak \
         /etc/yum.repos.d/fedora.repo \
         /etc/yum.repos.d/fedora-modular.repo \
         /etc/yum.repos.d/fedora-updates.repo \
         /etc/yum.repos.d/fedora-updates-modular.repo

# 更新本地缓存
sudo dnf makecache

# 更新软件
sudo dnf -y update
```



## 创建一个普通用户

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



## 配置窗口管理器

> xorg 服务器

```bash
# 安装依赖包
sudo dnf install xorg-x11-server-devel xorg-x11-xinit xorg-x11-server-Xorg

# 可选
xorg-x11-server-Xvfb xorg-x11-xauth xorg-x11-server-Xvfb xorg-x11-server-Xdmx xorg-x11-server-Xephyr xorg-x11-server-Xnest
```

在家目录下新建 `.xinitrc` 文件，并加入下面内容

```bash
#!/bin/bash
exec dwm
```



> dwm

```bash
# 安装依赖包
sudo dnf install cmake gcc libX11-devel libXft-devel libXinerama-devel git 

# 克隆 dwm
cd ~ && mkdir wm
git clone https://github.com/roukaixin/yaocccc-dwm

# 编译
sudo make clean install
```
**编译过程中，可能会缺失一些同文件，可以使用 `dnf provides */头文件名` 来查找在哪一个包**




> st

```bash
# 克隆 st
git clone https://gitee.com/rouxin/st

# 编译
sudo make clean install
```



> picom

```bash
# 安装依赖包
sudo dnf install meson libev-devel xcb-util-renderutil-devel xcb-util-image-devel pixman-devel xcb-util-devel uthash-devel libconfig-devel mesa-libGL-devel dbus-devel

# 克隆pciom
git clone https://github.com/FT-Labs/picom

# 编译
cd picom
git submodule update --init --recursive
meson setup --buildtype=release . build
ninja -C build
ninja -C build install
```



> 其他工具包

```bash
sudo dnf install network-manager-applet flameshot dunst feh neofetch neovim acpi xsetroot
```



> 安装字体

下载地址：`https://github.com/ryanoasis/nerd-fonts/releases/`

![image-20230714175135561](./note%20picture/fedoraLinux.assets/image-20230714175135561.png)

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







## 安装n卡驱动

**安装成功系统后，引导的时候需要开启第三方软件库**

```bash
sudo dnf install akmod-nvidia
```



## 安装docker

```bash
# 卸载之前安装的docker
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
# 安装存储库      
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
sudo sed -i 's+https://download.docker.com+https://mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo

# 安装 docker 引擎
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```



## 安装typora

```bash
# 这个可能用不了了
sudo dnf config-manager --add-repo https://typora.io/linux/Typora-fedora.repo
sudo dnf install typora
```
```bash
sudo dnf install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub io.typora.Typora
```





## 安装浏览器

> google chrome

```bash
# 下载
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
# 安装（推荐使用 dnf，会下载依赖库）
sudo dnf install ./google-chrome-stable_current_x86_64.rpm
```



> edge

```bash
# 导入GPG密钥
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
# 添加存储库
sudo dnf config-manager --add-repo https://packages.microsoft.com/yumrepos/edge
# 安装
sudo dnf install microsoft-edge-stable
```

---

> 作者: pankx  
> URL: https://roukaixin.github.io/fedora-linux/  

