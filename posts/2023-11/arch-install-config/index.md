# Arch Linux 基础安装到日常使用与美化 - 安装后配置



## 配置中国源和开启32位软件包

> 配置中国镜像源
```bash
# 编辑 /etc/pacman.d/mirrorlist，如果在安装系统时，关闭了 reflector 服务和配置了 mirrorlist，那么这个可以不用改，为了保险起见，还是需要看一下和安装的时候有没有区别。
vim /etc/pacman.d/mirrorlist

# 在头部添加中科大源
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```

{{< admonition info "提醒" true >}}
还可以添加其他源，例如：清华源(Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch)，阿里云（Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch）
{{< /admonition >}}

> 开启32位软件包
```bash
# 编辑 /etc/pacman.conf 
vim /etc/pacman.conf

# 下面两行内容取消掉 # 号
[multilib]
Include = /etc/pacman.d/mirrorlist
```

> 添加 aur 源
```bash
# 在最后面添加
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

{{< admonition info "提醒" true >}}
其他 aur 源。清华源（Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch），
阿里云源（Server = https://mirrors.aliyun.com/archlinuxcn/$arch）
{{< /admonition >}}

**必须执行以下命令以添加密钥签名**

```bash
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

![image-20230625065238368](images/Arch%20Linux.assets/image-20230625065238368.png " ")


{{< admonition info "提醒" true >}}
**我自己个人是注释掉 `%wheel ALL=(ALL:ALL)ALL` 这一行，方便之后使用 `sudo` 时不需要输入密码。**

**如果不想使用 `sudo` 命令时输入密码，可以注释掉 `%wheel ALL=(ALL:ALL) NOPASSWD: ALL`。两个选择一个注释掉既可以了**
{{< /admonition >}}


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
# fcitx5-pinyin-zhwiki : 中文维基百科创建的词库		
pacman -S fcitx5-pinyin-zhwiki fcitx5-pinyin-moegirl
```


## 安装工具（可选）

```bash
# udisk2 udiskie : 自动挂载U盘
# ntfs-3g ： 读取ntfs格式磁盘
# dolphin : 文件管理器
# thunar : 文件管理器
pacman -S udisks2 udiskie ntfs-3g  dolphin

# 开机自启自动挂载U盘
systemctl enable udisks2
```

{{< admonition info "提醒" true >}}
`systemctl enable udisks2` 根据个人是否开启，如果不开启，可以在启动窗口管理器的时候在启动也行
{{< /admonition >}}


---

> 作者:   
> URL: https://roukaixin.github.io/posts/2023-11/arch-install-config/  
