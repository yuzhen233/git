# 1：安装 archlinux

## 1：检查引导方式

```bash
ls /sys/firmware/efi/efivars
```

如果没有列出文件则不是`EFI引导+GPT分区表`引导方式

## 2：联网

```bash
dhcpcd
ping www.baidu.com
```

注意整个安装过程中重启系统后都应该执行联网步骤

## 3：更新系统时间

```bash
timedatectl set-ntp true
```

## 4：分区与格式化

```bash
fdisk -l
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

## 5：挂载分区

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## 6：选择镜像源

```bash
vim /etc/pacman.d/mirrorlist
```

将国内镜像源放置最前方即可

## 7：安装基本包

```bash
pacstrap /mnt base base-devel
```

## 8：配置Fstab

```bash
genfstab -L /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

## 9：Chroot

```bash
arch-chroot /mnt
```

## 10：设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

## 11：提前安装必须软件包

```bash
pacman -S vim dialog wpa_supplicant ntfs-3g networkmanager
```

## 12：中文化

### 1：安装中文locale

```bash
vim /etc/locale.gen
取消对应项前的注释符号「#」
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

locale-gen
```

### 2：启用中文locale

```bash
vim /etc/locale.conf
LANG=en_US.UTF-8
```

**单独在图形界面启用中文locale**

在`~/.bashrc`、`~/.xinitrc`、`~/.xprofile`里单独设置中文locale：

```bash
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
export LC_CTYPE=en_US.UTF-8
```

**图形界面用户设置全面的中文**

```bash
vim ~/.xprofile
export LC_ALL="zh_CN.UTF-8"
```

### 3：安装中文字体

```bash
pacman -S noto-fonts
```

## 13：设置主机名

```bash
vim /etc/hostname
输入主机名
vim /etc/hosts
127.0.0.1	localhost
::1		    localhost
127.0.1.1	myhostname.localdomain	myhostname
```

## 14：设置Root密码

```bash
passwd`
```

## 15：安装`Intel-ucode`

```bash
pacman -S intel-ucode
```

## 16：安装`Bootloader`

```bash
1：pacman -S os-prober
2：pacman -S grub efibootmgr
3：grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
4：grub-mkconfig -o /boot/grub/grub.cfg
5：vim /boot/grub/grub.cfg
6：exit reboot
```

## 17：创建交换文件

```bash
fallocate -l 512M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
vim /etc/fstab
/swapfile none swap defaults 0 0
```

## 18：新建用户

```bash
useradd -m -G wheel username
passwd username
```

## 19：配置sudo

```bash
pacman -S sudo
visudo
去掉之前的#注释符，保存并退出
# %wheel ALL=(ALL)ALL
reboot
```

## 20：显卡驱动的安装

```bash
sudo pacman -S xf86-video-intel
```

## 21：安装桌面环境

```bash
sudo pacman -S xorg
sudo pacman -S gnome
sudo pacman -S sddm
sudo systemctl enable sddm
```

## 22：提前配置网络

```bash
sudo systemctl disable netctl
sudo systemctl enable NetworkManager
```

# 2：**科学上网**

## 1：配置 SSR 客户端

```bash
pacman -S shadowsocks-qt5
```

​	附加免费`ssr`节点网址：`<https://github.com/Alvin9999/new-pac/wiki/ss%E5%85%8D%E8%B4%B9%E8%B4%A6%E5%8F%B7>`

## 2：使用 proxychains-ng

​	1：下载：

```bash
pacman -S proxychains-ng
```

​	2：配置：编辑 `/etc/proxychains.conf`

```bash
socks4 127.0.0.1 9095 改为：
socks5 127.0.0.1 1080
默认的socks4 127.0.0.1 9095是tor代理，而socks5 127.0.0.1 1080是shadowsocks的默认代理。
```

​	3：使用：

```bash
proxychains4 curl www.google.com
proxychains4 firefox
```

## 3：使用 Proxy SwitchyOmega

​	chrome 无法访问外网时，可以在`getcrx.cn`或者`github`上下载该`crx`文件，下载后讲该`crx`格式文件右键解压到某文件夹下，然后`chrome`打开扩展页面，点击开发者模式，然后加载已解压的扩展程序，选择该文件夹，即可安装该插件。