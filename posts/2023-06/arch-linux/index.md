# Arch Linux 安装到美化





# archlinux的基本安装

## 停止reflector服务 禁止自动更新服务器列表

```bash
systemctl stop reflector.service
```



## 检查电脑是否支持UEFI引导模式

```bash
ls /sys/firmware/efi/efivars
# 提示：如果没有报错就是支持UEFI
```



## 连接网络

### 有线连接

直接插上网线就可以自动联网了



### 无线网络（wifi）

```bash
# 是否启用了网络接口
ip link 
# 执行 iwctl 命令，进入交互式命令行
iwctl 
# 列出设备名，比如无线网卡看到叫 wlan0
device list   
# 用 wlan0 网卡扫描网络
station wlan0 scan 
# 列出网络
station wlan0 get-networks  
# 连接网络名字 输入密码
station wlan0 connect 无线网名字    
# 成功后退出
exit或者quit   
```

连接成功后ping一下百度是否有网

```bash
ping baidu.com
```



## 同步时间

```bash
# 同步网络时间
timedatectl set-ntp true
# 提示：检查是否成功 看到（system clock synchronized :yes）这一句就是成功了
timedatectl status
```


## 修改软件源

- 打中国的源放在头部

```bash
vim /etc/pacman.d/mirrorlist
```

- 36 dd : 剪切36行

- p  : 粘贴



## 分区

### fdisk分区

```bash
# 查看磁盘分区
lsblk
# 分区
fdisk  /dev/sda
```



### cfdisk分区

```bash
cfdisk /dev/sda
```

现在使用 `cfdisk` 分区有主分区和扩展分区，如果超过==4== 个主分区，那么就需要在扩展分区上进行分区。

例子：2G扩展分区（1G的 /boot 分区，1G 的 /boot/efi 分区），16G主分区（swap 交换分区），500G主分区 （ /home 分区），482G主分区（ 根分区）

**一般分区都有跟分区（/）、交换分区（swap）、引导分区（boot）**



### 格式化分区

> EFI 分区

两个选择其中一个

```bash
# mkfs.fat -F32 /dev/sda1 和 mkfs.vfat /dev/sda1
mkfs.vfat /dev/sda1
```



> swap 分区

```bash
mkswap /dev/sda2
```



> 普通分区

```bash
# 使用 ext4 格式格式化
mkfs.ext4 /dev/sda3

# 使用 btrfs 格式进行格式化
mkfs.btrfs -f -L arch /dev/sda3
```

- btrfs
  - `-f`  ：强制格式化为 `btrfs` 格式
  - `-L` ： 表示分区 `label` （可以自定义），可以理解为 `win` 下的磁盘名称。

 

## 挂载分区

- 根据自己的分区情况进行挂载分区（尽量不要把usr目录挂载出去，如果挂载出去是不能开机的（网上也有教程是可以挂载的，我没有试过）），一般挂载第三方应用安装目录（opt）、临时文件目录（tmp）


{{< admonition warning "注意" true >}}
  必须先挂载根目录才能挂载其他目录
{{< /admonition >}}


> 挂载根分区

-  `btrfs` 格式化挂载方法

    ```bash
    # 先挂载 /dev/sda3 分区。注意： 这个分区是 btrfs 分区，也是我的根分区
    mount -t btrfs -o compress=zstd /dev/sda3 /mnt
    
    # 创建两个子卷
    btrfs subvolume create /mnt/@
    btrfs subvolume create /mnt/@home
    
    # 查看创建的子卷，-p : 指定哪个分区
    btrfs subvolume list -p /mnt
    
    # 卸载 /dev/sda3 分区
    umount /mnt
    
    # 挂载根目录
    mount -t btrfs -o subvol=/@,compress=zstd /dev/sda3 /mnt
    
    # 挂载 home 目录
    mkdir /mnt/home
    mount -t btrfs -o subvol=/@home,compress=zstd /dev/sda3 /mnt/home
    
    # 挂载 EFI 分区
    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot
    
    # 挂载 swapon 分区
    swapon /dev/sda2
    ```

    {{< admonition warning "警告" true >}}快照工具 `timeshift` 只支持 /@ 这种子卷布局，如果采用不同的布局，`thimeshift` 可能会存在问题。
    {{< /admonition >}}




-  `ext4`  格式化挂载方法

    ```bash
    # 挂载 
    mount /dev/sda3 /mnt
    swapon /dev/sda2  请用swap分区
    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot
    ```




## 安装内核

```bash
# 往/mnt目录里安装系统
# 其中最基础的四个包是 base base-devel linux linux-firmware
# linux-lts （lts:稳定版）
# 如果内核安装了稳定版，那么独显也要是稳定版的，要不然就会发生问题（我也不知道什么问题）
pacstrap /mnt base base-devel linux linux-firmware vim
```

![image-20220924005039396](./note%20picture/Arch%20Linux.assets/image-20220924005039396.png " ")

## 配置系统

### Fstab

```bash
# 生成 fstab文件 (用 `-U` 或 `-L` 选项设置 UUID 或卷标)
genfstab -U /mnt >> /mnt/etc/fstab
```

{{< admonition info "强烈建议" true >}}在执行完以上命令后，检查一下生成的 `/mnt/etc/fstab` 文件是否正确。{{< /admonition >}}



### Chroot

```bash
# chroot到新安装的系统
arch-chroot /mnt
```



### 设置时区

```bash
# ln -sf /usr/share/zoneinfo/Region（地区名）/City（城市名） /etc/localtime
# 设置上海时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 将硬件时钟调整为与目前的系统时钟一致
hwclock --systohc
```



### 本地化

> 修改 local.gen 文件

程序和库如果需要本地化文本，都依赖区域设置，后者明确规定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

需在这两个文件设置：`locale.gen` 与 `locale.conf`。

编辑 `/etc/locale.gen`，然后取消掉 `en_US.UTF-8 UTF-8` 和其他需要的 区域设置前的注释#。

接着执行 `locale-gen` 以生成 locale 信息：

```bash
vim /etc/locale.gen
# 更新locale
locale-gen
```

> 创建 locale.conf 文件

创建 locale.conf 文件，并编辑设定 LANG 变量，比如：

```
vim /etc/locale.conf
LANG=en_US.UTF-8
```

**警告：** 不推荐在此设置任何中文 locale，会导致 tty 乱码。

例子：

```bash
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_SG.UTF-8 UTF-8
```



### 网络配置

> 创建hostname文件
>

```bash
# 编辑 hostname，写入下面内容
vim /etc/hostname

# 写入内容
arch（主机名）
```

> 创建hosts文件
>

```bash
# 编辑 hosts，写入下面内容
vim /etc/hosts

# 写入内容
127.0.0.1   localhost
::1         localhost
```



### Root 密码

```bash
# 修改 root 密码
passwd
```



## 安装引导程序

### 安装 cpu微码和引导软件

```bash
pacman -S intel-ucode  grub efibootmgr os-prober
# 如果是intel的cpu 则输入amd-ucode
# os-prober查找已安装的操作系统 推荐实体机上安装
```



### 安装grub引导

```bash
#安装grub引导
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

**说明**

- grub-install：`安装grub`
- --target=x86_64-efi ：`目标架构x86架构64位 efi启动方式  若不确定使用uname -a 可以查看`
- --efi-directory=/boot ：`安装系统是 efi 分区挂载的文件目录。例如：我 efi 挂载到 /boot 目录下，这里就是 /boot，如果是 /boot/efi 目录下，那这里就要写成 /boot/efi`
- --bootloader-id=ARCH ：`ARCH 是引导的别名，甚至这一段都可以不用写`



### 生成grub

```bash
# 生成grub ，如果没有 /boot/grub 目录就创建。（创建目录：mkdir /boot/grub）
grub-mkconfig -o /boot/grub/grub.cfg
```



## 安装软件

```bash
# 连接网络相关的包，如果安装了 networkmanager 就不需要安装 dhcpd 。如果你使用的是 dhcpcd 那就需要结合 iwd 来使用 wifi，networkmanager 不需要 iwd。
dhcpcd iwd	networkmanager
# 命令补全工具
bash-completion  
# 网络工具
iproute2 
#
zsh
# 查看系统信息命令
neofetch  

# 安装  
pacman -S networkmanager neofetch bash-completion

# 开机自启动网络管理。如果安装的是 dhcpd ，执行 systemctl enable dhcpcd
systemctl enable NetworkManager
```



## 退出

```bash
#输入 exit 或按 Ctrl+d 退出 chroot 环境。
exit
#卸载被挂载的分区
umount -R /mnt
#重启
reboot
```





# 安装后的配置

## 配置中国源和开启32位软件包

```bash
# 编辑 /etc/pacman.d/mirrorlist，如果在安装系统时，关闭了 reflector 服务和配置了 mirrorlist，那么这个可以不用改，为了保险起见，还是需要看一下和安装的时候有没有区别。
vim /etc/pacman.d/mirrorlist

# 清华源
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
# 中科大源
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
# 在头部添加阿里源
Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch
# 163源 （i686 和 x86_64）
Server = https://mirrors.163.com/archlinux/$repo/os/i686

################ 开启32位软件包 ################
# 编辑 /etc/pacman.conf 
vim /etc/pacman.conf

# 下面两行内容取消掉 # 号
[multilib]
Include = /etc/pacman.d/mirrorlist
# 在最后面添加
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.aliyun.com/archlinuxcn/$arch


# 更新软件包缓存
pacman -Syy
# 安装 GPG key
pacman -S archlinuxcn-keyring 
```



## 添加一个普通用户

```bash
#新建用户名arch 可自行更改用户名
useradd -m -G wheel -s /bin/bash arch

#设置arch用户名的密码
passwd arch

#编辑arch用户的权限
EDITOR=vim visudo

找到 # %wheel ALL=(ALL:ALL)ALL 并把 # 号去掉
```

![image-20230625065238368](./note%20picture/Arch%20Linux.assets/image-20230625065238368.png " ")

**个人是注释掉 `%wheel ALL=(ALL:ALL)ALL` 这一行，如果不想使用 `sudo` 命令时输入密码，可以注释掉 `%wheel ALL=(ALL:ALL) NOPASSWD: ALL`。两个选择一个注释掉既可以了**



## 安装软件源（AUR）

```bash
# 可以选择两个都安装，也可以只安装一个
pacman -S yay paru
```



## 安装字体

```bash
# ttf-dejavu : 英文字体
# wqy-zenhei : 文泉驿正黑矢量字体
# wqy-microhei : 文泉驿-微米黑
# noto-fonts-emoji : Emoji 字体
# ttf-jetbrains-mono-nerd :
pacman -S ttf-dejavu  wqy-zenhei wqy-microhei noto-fonts-emoji
```



## 安装中文输入法 

> 安装 fcitx5

```bash
# fcitx5-im : 基础包和模块
# fcitx5-chinese-addons ：中文输入法引擎
# 
# 输入法模块 1、qt程序：fcitx5-qt 2、gtk程序 ： fcitx5-gtk
pacman -S fcitx5-im  fcitx5-chinese-addons
```

> 配置输入法
>

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

> 安装词库

```bash
# 安装词库
# fcitx5-pinyin-zhwiki : 中文维基百科创建的词库		
pacman -S fcitx5-pinyin-zhwiki fcitx5-pinyin-moegirl
```

**全部配置完成之后重启一下电脑，使用普通用户登录，在进行下面操作**



## 安装工具（可选）

```bash
# udisk2 udiskie : 自动挂载U盘修改
# ntfs-3g ： 读取ntfs格式磁盘
# dolphin : 文件管理器
# thunar : 文件管理器
pacman -S udisks2 udiskie ntfs-3g  dolphin

# 开机自启自动挂载U盘
systemctl enable udisks2
```





# 显卡驱动

## 安装 NVIDIA 驱动

> 安装驱动

安装启动之前，先跟新一下 pacman 数据库。更新命令：`sudo pacman -Syyu`

```bash
# 安装 linux-lts-headers，因为显卡启动需要。
# 注意：如果内核是 linux ，不是 linux-lts，那么安装的是 linux-headers
sudo pacman -S linux-lts-headers
```

```bash
# 安装 nvidia 启动，推荐安装 nvidia-dkms ，他是动态安装的。不像其他驱动，如果内核升级，就需要重新安装一次驱动。
sudo pacman -S nvidia-dkms
```



> DRM 内核模式设置

**这一步可以解决屏幕撕裂问题**

```bash
# 编辑修改 /etc/default/grub
vim /etc/default/grub

# 找到 GRUB_CMDLINE_LINUX_DEFAULT 这一项在后面加入 nvidia-drm.modeset=1
```

![image-20230507044757042](./note%20picture/Arch%20Linux.assets/image-20230507044757042.png " ")

![image-20230506101749305](./note%20picture/Arch%20Linux.assets/image-20230506101749305.png " ")



**编辑完成之后，重新生成 `grub.cfg` ，执行 `sudo  grub-mkconfig -o /boot/grub/grub.cfg`**



> 添加 initramfs 模块

在 `/etc/mkinitcpio.conf` 中找到  `MODULES=` 这一行在括号的后面加入`nvidia nvidia_modeset nvidia_uvm nvidia_drm`。修改之前先备份一下（`sudo cp /etc/mkinitcpio.conf /etc/mkinitcpio.conf.back`）

![image-20230507045036585](./note%20picture/Arch%20Linux.assets/image-20230507045036585.png " ")

> 重新生成 `initramfs `

```bash
# 输入 sudo mkinitcpio -P 命令生成 initramfs
sudo mkinitcpio -P
```



**重要：执行完命令之后重启电脑（一定要重启）**



> 校验是否安装成功

打开终端，输入 `nvidia-smi` 命令，出现 `NVIDIA-SMI has failed because it couldn't communicate with the NIVIDIA driver. Make sure that the latest NVIDIA driver is installed and running`  ，那就说明没有安装成功。

**解决方案：重启电脑，如果重启电脑还是不行，看看上面修改的配置有没有对，如果还是不行在安装一遍**



### 仅使用 NVIDIA 显卡

**下面配置针对于 x 服务器，并使用桌面环境（dwm）**

> 创建 xorg.conf（10-nvidia-drm-outputclass.conf）

在 `/etc/X11/xorg.conf.d/` 目录下新建 `10-nvidia-drm-outputclass.conf`。

**重要：如果`/etc/X11`存在`xorg.conf`，那么一定要删除掉**

在`/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf`也有一份是生成的，可以参考着来看

```bash
vim /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf

Section "OutputClass"
    Identifier "intel"
    MatchDriver "i915"
    Driver "modesetting"
EndSection

Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "PrimaryGPU" "yes"
    ModulePath "/usr/lib/nvidia/xorg"
    ModulePath "/usr/lib/xorg/modules"
EndSection
```

**注意：** 这里的  `intel`  驱动 **必须为** `modesetting`，否则会产生严重的撕裂，后面的 `AllowEmptyInitialConfiguration` 是为了让 n 卡在没检测到显示器的时候继续运行，而不是默认的退出。



> 使用 startx 启动

这里以**dwm**为例，如果你是其他桌面管理器，请参阅[这里](https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Display_Managers)。如果不配置你将会获得黑屏。 

接下来，将以下两行添加到您的开头：`~/.xinitrc`

```bash
# 编辑 .xinitrc 并在文件开头添加下面内容
vim ~/.xinitrc

xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

![image-20230507052853861](./note%20picture/Arch%20Linux.assets/image-20230507052853861.png " ")

**如果启动之后 dpi 设置不正确，那么还需要在 `xrandr --auto` 下面一行加入 `xrandr --dpi 96`**



> 安装 NVIDIA 软件包 （可选）

```bash
sudo pacman -S nvidia-settings nvidia-utils
```

![image-20230507052136496](./note%20picture/Arch%20Linux.assets/image-20230507052136496.png " ")



> 检查3D

检查是否正常使用 N 卡来输出。建议检查一下

```bash
# 安装包
sudo pacman -S mesa-utils

# 检查是否输出,出现下面内容就是成功了
glxinfo | grep NVIDIA
```

![image-20230507053421591](./note%20picture/Arch%20Linux.assets/image-20230507053421591.png " ")



#### 配置视频硬解

```bash
# 安装 翻译层
paru -S libva-nvidia-driver
```



> 配置

方法一：

```bash
# 创建 .xprofile 文件
vim ~/.xprofile

export LIBVA_DRIVER_NAME=nvidia
export LIBVA_DRIVERS_PATH=/usr/lib/dri/
export VDPAU_DRIVER=nvidia

# 修改 .xinitrc 文件
[ -f ~/.xprofile ] && . ~/.xprofile
```

![image-20230507053717011](./note%20picture/Arch%20Linux.assets/image-20230507053717011.png " ")

方法二：

```bash
# 修改 .xinitrc 文件
export LIBVA_DRIVER_NAME=nvidia
export LIBVA_DRIVERS_PATH=/usr/lib/dri/
export VDPAU_DRIVER=nvidia
```

![image-20230625081349283](./note%20picture/Arch%20Linux.assets/image-20230625081349283.png " ")

**注意：一定要在 `exec dwm` 之前**



> 验证 VA-API

```bash
# 安装软件包，如果不安装这个包，那么 vainfo 无法使用
sudo pacman -S libva-utils
```

验证 VA-API 是否正常工作，运行 `vainfo`，出现下面内容就表示成功



![image-20230507055147711](./note%20picture/Arch%20Linux.assets/image-20230507055147711.png " ")

> 验证VDPAU

```bash
# 安装 vdpauinfo，不安装无法使用 vdpauinfo 命令
sudo pacman -S vdpauinfo
```



验证 VDPAU 是否正常工作，运行`vdpauinfo`，出现下图就表示成功

![image-20230507055418720](./note%20picture/Arch%20Linux.assets/image-20230507055418720.png " ")



# 声卡驱动

```bash
sudo pacman -S pipewire
sudo pacman -S lib32-pipewire
sudo pacman -S pipewire-pulse
sudo pacman -S pavucontrol
```





# wm（窗口管理器）

## dwm（窗口管理器）的安装

### 安装x窗口管理系统

```bash
# xorg  ： xorg 包含 xorg-server 和 xorg-apps
# xorg-xinit ： 用来启动xorg
sudo pacman -S xorg xorg-xinit

# 安装完xorg-xinit之后把配置文件复制到普通用户目录下
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```



### 安装dwm全家桶

#### 安装dwm

```bash
# 下载dwm
git clone https://git.suckless.org/dwm
cd dwm
# 编译
make
# 安装
make install
```



#### 安装st

```bash
# 下载st （alacritty 终端）
git clone https://git.suckless.org/st
cd st
# 编译
make
# 安装
make install
```



#### 安装dmenu

```bash
# 下载dmenu （rofi）
git clone https://git.suckless.org/dmenu
cd qmenu
# 编译
make
# 先清除在安装
make clean install
```



#### 安装slstaus

```bash
# 下载安装 slstatus
git clone https://git.suckless.org/slstatus
cd slstatus
make
make clean install

# 在 ~/.xinitrc 中加入 exec slstatus & 。注意：一定要在 dwm 前面
```

![image-20220925144617614](./note%20picture/Arch%20Linux.assets/image-20220925144617614.png " ")



#### 启动

#### 设置中文界面

```bash
# 在 ～/.xintrx 中加入
# 注意 ： 一定要加在 exec dwm 前
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```



#### 启动dwm

```bash
# 编辑 .xinitrc 文件
vim ~/.xinitrc

# 先把最后几行删除，在把 exec dwm 添加进去
```


**在`～/.xinitrc`文件末尾添加`exec dwm`，并保存退出，使用`startx`来启动窗口管理器。** 



#### 设置分辨率

```bash
# 查看分辨率
xrandr -q

# 设置分辨率
# Virtual-1  : 显示器名称
# 1920x1080  ： 分辨率
# 144.00 ： 赫兹 
xrandr --output Virtual-1 --mode 1920x1080 --rate 144.00 
```

注意：如果使用`xrandr -q`查看的结果中没有符合自己显示的分辨率就需要自己创建一个

```bash
# 创建一个分辨率
cvt 1920 1080 144
xrandr --newmode ....
# 添加到显示器上
xrandr --addmode 显示器名称 "1920x1080_144.00"
# 修改分辨率
xrandr --output 显示器名称 --mode '分辨率'
```

![image-20220928004105215](./note%20picture/Arch%20Linux.assets/image-20220928004105215.png " ")



# picom合成器

```bash
# 安装 picom 、picom-ibhagwan-git(毛玻璃效果)、picom-ftlabs-git（动画效果很好）
paru -S picom-ftlabs-git

# 虚拟机下要是 picom 生效要进行配置需要注释掉 vsync= true
# ~/.config/picom/picom.conf
mkdir ~/.config/picom
# 如果没有 pciom.conf ，那么就看看有没有 picom.conf.example
cp /etc/xdg/picom.conf ~/.config/picom/picom.conf
vim /etc/xdg/picom.conf


# 启动 dwm 启动 picom
# -b: 以后台进程（Daemon）的形式运行
# -c: 启用阴影效果
# -C: 禁用面板和 Docks 的阴影效果
# -G: 禁用应用程序窗口和拖放对象的阴影效果
# --config: 使用指定的配置文件
vim ~/.xinitrc
picom --config ~/wm/config/picom/picom.conf
# 添加在 exec dwm 之前
```

**注意**： 基本都是把全局的配置文件负责到当前用户下的`.config/picom`目录下。命令：`cp /etc/xdg/picom.conf ~/.config/picom/picom.conf`



**class_g 怎么获取：**

- **1、下载安装 `xorg-xprop` **
- **2、在终端里运行 `xprop` 命令，之后点击需要获取的 `class` 的窗口** 



> 排除透明度的程序

```bash
focus-exclude = [
	"class_g = 'fcitx'"
]
# 表示排除选中的透明度


opacity-rule = [
	"100:class_g = 'fcitx'"
]
# 表示 fcitx 的不透明度
# 其中 100 表示不透明度为 100%
```

![image-20230507064748371](./note%20picture/Arch%20Linux.assets/image-20230507064748371.png " ")

> 开启圆角

```bash
# 圆角大小，值大于0
corner-radius = 10
# 排除圆角程序
rounded-corners-exclude = [
	"class_g = 'fcitx'"
]
```



![image-20230507065032647](./note%20picture/Arch%20Linux.assets/image-20230507065032647.png " ")

**我的配置文件地址：`https://gitee.com/rouxin/config`**





# 系统优化



## 提升开机速度

```bash
# 事后可设置 /etc/default/grub 中 可提升启动速度
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 nowatchdog"
# 重新生成
grub-mkconfig -o /boot/grub/grub.cfg
```


## 双系统时间不正确

windows使用 UTC 时间

```bash
# 进入 windows 系统，使用管理员运行 cmd，并运行下面命令于添加到注册表中
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

## 调节亮度

**方法一：**

默认亮度：24000

> 查看最大亮度

```bash
cat  /sys/class/backlight/intel_backlight/max_brightness
```

> 修改亮度值

```bash
sudo vim /sys/class/backlight/intel_backlight/brightness
```



**方法二：**

```bash
# 安装 brightnessctl 包
sudo pacman -S brightnessctl
```

在 dwm 的 config.h 中加入

```bash
static const char *brighter[] = { "brightnessctl", "set", "10%+", NULL };
static const char *dimmer[]   = { "brightnessctl", "set", "10%-", NULL };
...

static const Key keys[] = {
       ...
       
       { 0, XF86XK_MonBrightnessDown, spawn, {.v = dimmer } },
       { 0, XF86XK_MonBrightnessUp,   spawn, {.v = brighter } },
       
       ...
};
```







## 打补丁方法



进行下面操作之前需要配置好 `git`


```bash
# 下载 git ，推荐使用git打补丁，可以进行回退配置
# kdiff3 : 对比 git 冲突的图形化页面
pacman -S git kdiff3
# 配置用户名和邮箱
git config --global user.name roukaixin
git config --global user.email "a3427173515@163.com"
# 设置全局查看分支冲突软件
git config --global merge.tool kdiff3
```





> 方法一
>

- 下载补丁

- git 操作

```bash
# 查看当前在那个分支
git branch
# 创建一个分支 config : 分支名
git branch config
# 切换分支
git checkout config
# 添加文件
git add 文件名
# commit
git commit -m 描述信息
# 融合分支 config :分支ming 123：描述信息
git merge config -m 123

# 清除到原始的状态
make clean && rm -rf config.h && git reset --hard origin/master
```



```bash

# 打补丁的命令
# -F 0  取消模糊匹配
patch  -F 0 补丁名字
  
# 打补丁失败的是生成一个  .dej 为结尾的文件，要自己手动去决定
vim .rej
# 打开一个vim的tab栏
:tabnew
# 打开（文件）
:e 文件名

```

  - 在.rej 文件里，前端有 + 号的就是表示要加进去，- 号就表示删除掉
  - 注意：函数的函数名不要删除掉





> 方法二

下载补丁，然后自己手动去复制粘贴到源代码里面



## dwm

进行到这一步的时候，因为之前已经编译了 dwm ，所以先重置回到官方的版本。`命令：make clean && rm -rf config.h && git reset --hard origin/master`

```bash
# 创建一个本地 sockless 分支
git branch sockless

# 删除官方的仓库地址并重新添加，下图第一个
git remote -v
git remote remove origin
git remote add sockless https://git.suckless.org/dwm

# 添加自己的仓库地址（这里使用的是两个仓库，分别是 gitee，github。自己根据需要来执行操作）
git remote add gitee git@gitee.com:rouxin/dwm.git
git remote add github git@github.com:roukaixin/dwm.git
# 出现下面第二张图说明就成功了

```

![image-20230507075646997](./note%20picture/Arch%20Linux.assets/image-20230507075646997.png " ")

![image-20230507080205654](./note%20picture/Arch%20Linux.assets/image-20230507080205654.png " ")



> 把官方的 dwm 源码分别提交到 gitee 和 github 仓库

```bash
# 切换到 sockless 本地分支
git checkout sockless
# 分别提交到 gitee 和 github 仓库的 sockless 分支
git push gitee sockless:sockless
git push github sockless:sockless

# 切换会 master 分支
git checkout master
# 提交到主分支
git push gitee master:master
git push github master:main
```

这些操作只是为了到时候升级dwm的官方版本比较方便，直接在 `sockless` 分支拉取最新的代码就可以了，之后在到 主分支和并 `sockless` 分支。

```bash
# 拉取
git pull sockless master:sockless
# 在主分支上合并 sockless 分支
git merge sockless -m "更新 dwm 官方代码"
```



### dwn推荐的补丁

- 窗口全屏（actualfullscreen）

- 透明补丁（alpha）
- 浮动布局，打开窗口位置居中（alwayscenter）
- 打开新窗口是在底部（attachbottom）
- 自启动脚本（autostart）
- 状态栏显示多个窗口信息（awesomebar）
- dwm 的 bar 高度（bar height）
- 屏幕和状态栏的间隙（barpadding）
- 网格布局（gridmode）
- 隐藏空的 tag （hide vacant tags）
- 移动窗口补丁 （movethrow）
- 每个 tags 都是独立的布局（pertag）
- 临时小窗口（scratchpad）
- 状态栏托盘（systray）
- 移动窗口到其他 tag ，跟随过去（viewontag）
- 窗口间距（vanitygaps）

其他补丁可以参考`https://toscode.gitee.com/xiexie1993/dwm` git 仓库



### 布局图标

```bash
# 地址
https://www.nerdfonts.com/
# 网格布局图标
nf-md-view_grid
# 平铺布局图标
nf-md-collage
```



### 修改关键的键为win键

```bash
cd dwm
vim config.h
#修改
#define MDDKEY MOD4Msk
```



![image-20220924013208040](./note%20picture/Arch%20Linux.assets/image-20220924013208040.png " ")



### 设置中英文字体大小相同

```bash
# 安装矢量图标
pacman -S ttf-nerd-fonts-symbols-2048-em

# 进入dwm目录下编写 config.h 文件
cd dwm 
vim config.h
# 查看文泉驿字体 
fc-list |grep WenQuanYi

# 在config.h中添加配置
# antialias autohint : 抗锯齿
“WenQuanYi Micro Hei:size=10:type=Regular:antialias=true:autohint=true”

# 查看矢量图标
fc-list |grep Nerd
“Symbols Nerd Font:pixelsize=14:type=2048-em:antialias=true:autohint=true”
```

![image-20220924233710569](./note%20picture/Arch%20Linux.assets/image-20220924233710569.png " ")



![image-20221001210909598](./note%20picture/Arch%20Linux.assets/image-20221001210909598.png " ")



### 设置tag图标

```bash
# 图标网址
https://www.nerdfonts.com/cheat-sheet

# terminal  终端
# google  浏览器
# java   java编程
# markdown 笔记
# folder  文件管理
# office 文档
# music  音乐
# steam  游戏
# cube   虚拟机

# video  录屏



# 设置完成后重新编译安装
make && sudo make clean install
```

![image-20220925134618485](./note%20picture/Arch%20Linux.assets/image-20220925134618485.png " ")

![image-20220925134649175](./note%20picture/Arch%20Linux.assets/image-20220925134649175.png " ")



### 设置状态栏

**使用音量要安装alsa-utils包** ：`pacman -S  alsa-utils`

```bash
# ==============图标==================#
# 启动版本号
archlinux
# 硬盘
disk
# cpu 
memory
# 内存
nf-mdi-chip
nf-fa-microchip
# 上传网速
nf-mdi-arrow_up_bold
# 下载网速
nf-mdi-arrow_down_bold
# 音量
volume
# 麦克风
microphone
# 时间
time
#=====================================#
#==================命令===================#
如图片
```

![image-20221001202825882](./note%20picture/Arch%20Linux.assets/image-20221001202825882.png " ")





### 设置快捷键

#### 设置音量快捷键

```bash
# 切换音量状态（静音、非静音） 
amixer sset Master toggle

# 最小步长是3,设置5一下都是以3个音量增加或减少
# 减少音量
amixer sset Master 5%- unmute
# 增大音量
amixer sset Master 5%+ unmute
```



```bash
# 进入 dwm 的目录
cd dwm
# 创建存放脚本的目录
mkdir script
# 进入 script 目录
cd script
# 编写加音量的脚本
vim volup.sh
#! /bin/bash
amixer sset Master 3%+ unmute
# 编写减音量的脚本
vim voldown.sh
#! /bin/bash
amixer sset Master 3%- unmute
# 编写切换音量状态的脚本
vim voltoggle.sh
#! /bin/bash
amixer sset Master toggle

# 编写完脚本之后给脚本可执行的权限
chmod +x volup.sh voldown.sh voltoggle.sh

# 注意 ： 把 script 目录归到普通用户组
```












## st

### st推荐补丁

- 终端半透明（alpha）
- 去除终端的白边（anysize） 
  - 打开多个终端就会出项这个问题
- 终端中的输入下划线（blinking_cursor）
  - 闪动的下划线
- 主题颜色（xresources）
  - 保留多个主颜色





### 修改st的字体大小

```bash
cd st
vim config.h
#修改
```



![image-20220924013544614](./note%20picture/Arch%20Linux.assets/image-20220924013544614.png " ")



## dmenu



## slstaus







## 设置背景图片

```bash
# feh 图形工具
pacman -S feh

# 设置壁纸
# --bg-fill : 全屏显示
# --randomize ：随即显示
feh --bg-fill --randomize ~/wallpaper/*.png(壁纸路径)

# 设置启动 dwm 应用壁纸
vim ~/.xinitrc
feh --bg-fill --randomize ~/wallpaper/*.png(壁纸路径)

# 刷新背景图片
while feh --bg-fill --randomize ~/wallpaper/*.png(壁纸路径)
do
	sleep 60   # 睡多少秒
done &
```

![image-20220924120805094](./note%20picture/Arch%20Linux.assets/image-20220924120805094.png " ")









# 软件安装

## 网络管理器

```bash
# 系统托盘
sudo pacman -S network-manager-applet
```



> 开启热点

```bash
# 安装 linux-wifi-hotspot
paru -S linux-wifi-hotspot
# 开启热点
# 第一个 wlo1 : 热点连接
# 第二个 wlo1 : 共享的网络数据
# arch ： 热点名字
# 12345678 ： 热点密码
sudo create_ap wlo1 wlo1 arch 12345678
```





## 截图软件

```bash
# 火焰截图
sudo pacman -S flameshot 
```

绑定 `dwm` 快捷键，在 `config.def.conf` 中加入 `{ Mod1Mask,                     XK_a,      spawn,          SHCMD("flameshot gui") },    //截图` （快捷键：alt + a）



## 通知守护进程

```bash
sudo pacman -S dunst

# 启动
dunst -config 文件路径
```



## 终端文件管理器

```bash
sudo pacman -S ranger
```

官方文档：`https://kgithub.com/ranger/ranger/wiki`

其他文档配置：`https://www.zssnp.top/2021/06/03/ranger/`

### 生成配置文件

```bash
ranger --copy-config=all
```

![image-20230525000537299](./note%20picture/Arch%20Linux.assets/image-20230525000537299.png " ")

### 安装图标

```bash
git clone https://github.com/alexanderjeurissen/ranger_devicons ~/.config/ranger/plugins/ranger_devicons
echo "default_linemode devicons" >> $HOME/.config/ranger/rc.conf
```



### 预览图片

```bash
# 安装 ueberzugpp
paru -S ueberzugpp

# 修改配置。～/.config/ranger/rc.conf
set preview_images false 改为 set preview_images true
set preview_images_method w3m 改为 set preview_images_method ueberzug
```



### 预览视频

```bash
# 安装 ffmpegthumbnailer
sudo pacman -S ffmpegthumbnailer
# 修改配置文件。～/.config/ranger/scope.sh
取消掉 video 的注释
```





![image-20230525011949986](./note%20picture/Arch%20Linux.assets/image-20230525011949986.png " ")



### 代码高亮

```bash
sudo pacman -S highlight
```



### 压缩包预览

```bash
sudo pacman -S atool
```





## 文档软件

### wps

```bash
paru -S wps-office
paru -S ttf-wps-fonts
paru -S wps-office-mui-zh-cn
```



## 代理

### clash

> 下载配置文件

```bash
wget -o confing.yaml 订阅地址&flag=clash
```



> docker compose 

```yaml
version: "3.8"

services:
  clash:
    image: ghcr.io/dreamacro/clash
    restart: always
    volumes:
      - /posts/config/config.yaml:/root/.config/clash/config.yaml:ro
    dns:
      - 114.114.114.114
    ports:
      - "7890:7890"
      - "7891:7891"
      - "9090:9090" # The External Controller (RESTful API)
    network_mode: "bridge"
```



> 终端代理

```bash
sudo pacman -S proxychains-ng
```

修改配置文件

```bash
sudo vim /etc/proxychains.conf

# 在最后修改为
socks5	127.0.0.1	7891
```



> googe 使用代理

```bash
# google-chrome-stable： 谷歌的命理
# 127.0.0.1 ： 代理的服务器，7890 ： http 代理的端口
google-chrome-stable --proxy-server=127.0.0.1:7890
```





## 文件管理器

```bash
# thunar
# pcmanfm
sudo pacman -S thunar
```



## 音乐播放器

### qq音乐

```bash
paru -S qqmusic-bin
```



### listen1

地址：`https://github.com/listen1/listen1_desktop`

![image-20230608194903103](./note%20picture/Arch%20Linux.assets/image-20230608194903103.png " ")

这里我选择的是 `AppImage` 格式，如果想在 rofi 中可以打开，那么要先给这个文件可执行的权限，并把他软链接到 `/usr/local/bin` 目录下。

**注意：一定要使用绝对路径，要不然找不到文件**







## 聊天软件

### 纸飞机

```bash
sudo pacman -S telegram-desktop
```



### qq

```bash
paru -S linuxqq
```



### 微信

```bash
paru -S com.qq.weixin.deepin

# 启动命令路径
/opt/apps/com.qq.weixin.deepin/files/run.sh

# 软连接到 /usr/local/bin
ln -sf /opt/apps/com.qq.weixin.deepin/files/run.sh /usr/local/bin/weixin
```

**注意：如果使用有那些框框，可以试着改一下启动命令的一些东西，本人使用 deepin-wine5 是比较好用的，还有一些 wine 的容器（这个需要自己去了解）**

![image-20230608195843418](./note%20picture/Arch%20Linux.assets/image-20230608195843418.png " ")





## 笔记软件

### typora

```bash
sudo pacman -S typora
```



## 录屏软件

### obs

```bash
sudo pacman -S obs-studio
```



## redis 可视化

```bash
paru -S redisinsight
```



> 官方地址

```bash
https://redis.com/redis-enterprise/redis-insight/

# 下载 appimager，给他赋予可执行权限，并把它软拦截到 /usr/local/bin 目录下
```



---

> 作者: pankx  
> URL: https://roukaixin.github.io/posts/2023-06/arch-linux/  

