# Arch Linux 基础安装到日常使用与美化 - 驱动安装



## 安装 NVIDIA 驱动

&gt; 安装驱动

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



&gt; DRM 内核模式设置

**这一步可以解决屏幕撕裂问题**

```bash
# 编辑修改 /etc/default/grub
vim /etc/default/grub

# 找到 GRUB_CMDLINE_LINUX_DEFAULT 这一项在后面加入 nvidia-drm.modeset=1
```

![image-20230507044757042](arch-drive-install.assets/image-20230507044757042.png &#34; &#34;)

![image-20230506101749305](arch-drive-install.assets/image-20230506101749305.png &#34; &#34;)



**编辑完成之后，重新生成 `grub.cfg` ，执行 `sudo  grub-mkconfig -o /boot/grub/grub.cfg`**



&gt; 添加 initramfs 模块

在 `/etc/mkinitcpio.conf` 中找到  `MODULES=` 这一行在括号的后面加入`nvidia nvidia_modeset nvidia_uvm nvidia_drm`。修改之前先备份一下（`sudo cp /etc/mkinitcpio.conf /etc/mkinitcpio.conf.back`）

![image-20230507045036585](arch-drive-install.assets/image-20230507045036585.png &#34; &#34;)

&gt; 重新生成 `initramfs `

```bash
# 输入 sudo mkinitcpio -P 命令生成 initramfs
sudo mkinitcpio -P
```



**重要：执行完命令之后重启电脑（一定要重启）**



&gt; 校验是否安装成功

打开终端，输入 `nvidia-smi` 命令，出现 `NVIDIA-SMI has failed because it couldn&#39;t communicate with the NIVIDIA driver. Make sure that the latest NVIDIA driver is installed and running`  ，那就说明没有安装成功。

**解决方案：重启电脑，如果重启电脑还是不行，看看上面修改的配置有没有对，如果还是不行在安装一遍**



### 仅使用 NVIDIA 显卡

**下面配置针对于 x 服务器，并使用桌面环境（dwm）**

&gt; 创建 xorg.conf（10-nvidia-drm-outputclass.conf）

在 `/etc/X11/xorg.conf.d/` 目录下新建 `10-nvidia-drm-outputclass.conf`。

**重要：如果`/etc/X11`存在`xorg.conf`，那么一定要删除掉**

在`/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf`也有一份是生成的，可以参考着来看

```bash
vim /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf

Section &#34;OutputClass&#34;
    Identifier &#34;intel&#34;
    MatchDriver &#34;i915&#34;
    Driver &#34;modesetting&#34;
EndSection

Section &#34;OutputClass&#34;
    Identifier &#34;nvidia&#34;
    MatchDriver &#34;nvidia-drm&#34;
    Driver &#34;nvidia&#34;
    Option &#34;AllowEmptyInitialConfiguration&#34;
    Option &#34;PrimaryGPU&#34; &#34;yes&#34;
    ModulePath &#34;/usr/lib/nvidia/xorg&#34;
    ModulePath &#34;/usr/lib/xorg/modules&#34;
EndSection
```

**注意：** 这里的  `intel`  驱动 **必须为** `modesetting`，否则会产生严重的撕裂，后面的 `AllowEmptyInitialConfiguration` 是为了让 n 卡在没检测到显示器的时候继续运行，而不是默认的退出。



&gt; 使用 startx 启动

这里以**dwm**为例，如果你是其他桌面管理器，请参阅[这里](https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Display_Managers)。如果不配置你将会获得黑屏。

接下来，将以下两行添加到您的开头：`~/.xinitrc`

```bash
# 编辑 .xinitrc 并在文件开头添加下面内容
vim ~/.xinitrc

xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

![image-20230507052853861](arch-drive-install.assets/image-20230507052853861.png &#34; &#34;)

**如果启动之后 dpi 设置不正确，那么还需要在 `xrandr --auto` 下面一行加入 `xrandr --dpi 96`**



&gt; 安装 NVIDIA 软件包 （可选）

```bash
sudo pacman -S nvidia-settings nvidia-utils
```

![image-20230507052136496](arch-drive-install.assets/image-20230507052136496.png &#34; &#34;)



&gt; 检查3D

检查是否正常使用 N 卡来输出。建议检查一下

```bash
# 安装包
sudo pacman -S mesa-utils

# 检查是否输出,出现下面内容就是成功了
glxinfo | grep NVIDIA
```

![image-20230507053421591](arch-drive-install.assets/image-20230507053421591.png &#34; &#34;)



#### 配置视频硬解

```bash
# 安装 翻译层
paru -S libva-nvidia-driver
```



&gt; 配置

方法一：

```bash
# 创建 .xprofile 文件
vim ~/.xprofile

export LIBVA_DRIVER_NAME=nvidia
export LIBVA_DRIVERS_PATH=/usr/lib/dri/
export VDPAU_DRIVER=nvidia

# 修改 .xinitrc 文件
[ -f ~/.xprofile ] &amp;&amp; . ~/.xprofile
```

![image-20230507053717011](arch-drive-install.assets/image-20230507053717011.png &#34; &#34;)

方法二：

```bash
# 修改 .xinitrc 文件
export LIBVA_DRIVER_NAME=nvidia
export LIBVA_DRIVERS_PATH=/usr/lib/dri/
export VDPAU_DRIVER=nvidia
```

![image-20230625081349283](arch-drive-install.assets/image-20230625081349283.png &#34; &#34;)

**注意：一定要在 `exec dwm` 之前**



&gt; 验证 VA-API

```bash
# 安装软件包，如果不安装这个包，那么 vainfo 无法使用
sudo pacman -S libva-utils
```

验证 VA-API 是否正常工作，运行 `vainfo`，出现下面内容就表示成功



![image-20230507055147711](arch-drive-install.assets/image-20230507055147711.png &#34; &#34;)

&gt; 验证VDPAU

```bash
# 安装 vdpauinfo，不安装无法使用 vdpauinfo 命令
sudo pacman -S vdpauinfo
```



验证 VDPAU 是否正常工作，运行`vdpauinfo`，出现下图就表示成功

![image-20230507055418720](arch-drive-install.assets/image-20230507055418720.png &#34; &#34;)



## 声卡驱动

```bash
sudo pacman -S pipewire
sudo pacman -S lib32-pipewire
sudo pacman -S pipewire-pulse
sudo pacman -S pavucontrol
```


## 蓝牙驱动

```bash
sudo pacman -S bluez bluez-utils blueman
```

启动蓝牙
```bash
systemctl start bluetooth.service
```

参考教程：`https://cn.linux-console.net/?p=16637`

---

> 作者: 不北咪  
> URL: https://roukaixin.github.io/posts/2023-11/arch-drive-install/  

