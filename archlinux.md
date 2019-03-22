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

**将简体中文设置为默认语言**

```bash
vim /etc/locale.conf
添加
LANG=zh_CN.UTF-8
```

由于对[Locale (简体中文)](https://wiki.archlinux.org/index.php/Locale_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))定义了框架内地区（即 CJK 优先度），使得默认的优先级被忽略。

### 3：安装中文字体

```bash
pacman -S noto-fonts-cjk
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
passwd
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
sudo pacman -S deepin deepin-extra
sudo pacman -S lightdm
sudo vim /etc/lightdm/lightdm.conf
找到该行去掉注释修改为以下代码
greeter-session=lightdm-deepin-greeter
sudo systemctl enable lightdm
```

## 22：提前配置网络

```bash
sudo systemctl disable netctl
sudo systemctl enable NetworkManager
```

# 2：安装完成后工作

## 1：zsh

```bash
chsh -l           查看系统自带哪些shell
echo $SHELL       查看当前环境shell

1：安装zsh
sudo pacman -S zsh
2：设置zsh为默认shell
chsh -s /usr/bin/zsh
3：重启
4：安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
5：配置oh-my-zsh
sudo vim ~/.zshrc
更改合适的主题`bira`
```

## 2：添加中文社区镜像源

```bash
sudo vim /etc/pacman.conf
添加以下代码
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

sudo pacman -Sy
sudo pacman -S archlinuxcn-keyring
sudo pacman -Sy
```

## 3：安装yay

```bash
在添加完中文社区镜像源后：
sudo pacman -S yay
```

## 4：安装搜狗输入法

```bash
sudo pacman -S fcitx
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5
sudo pacman -S fcitx-sogoupinyin
sudo vim ~/.xprofile
添加以下代码
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
注销重启，并打开`fcitx-configtool`设置快捷键
```

## 4：科学上网

### 1：配置 SSR 客户端

```bash
pacman -S shadowsocks-qt5
```

​	附加免费`ssr`节点网址：`https://github.com/Alvin9999/new-pac/wiki/ss%E5%85%8D%E8%B4%B9%E8%B4%A6%E5%8F%B7`

### 2：使用 proxychains-ng

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

### 3：使用 Proxy SwitchyOmega

首先尝试使用命令行进行chrome代理：

`````bash
google-chrome-stable --proxy-server="socks5://127.0.0.1:1080"
`````

​	上述办法无效或无法访问外网时，可以在`getcrx.cn`或者`github`上下载该`crx`文件，下载后讲该`crx`格式文件右键解压到某文件夹下，然后`chrome`打开扩展页面，点击开发者模式，然后加载已解压的扩展程序，选择该文件夹，即可安装该插件。

在原有的两个配置上进行更改：

`proxy`：更改为：`长城防火墙`

代理协议：socks5； 代理服务器：127.0.0.1； 代理端口：1080

------

`autoswitch`：更改为：`自动切换模式`

条件类型：`域名通配符`； 条件设置：`raw.githubusercontent.com`； 情景模式：`长城防火墙`； 

点击`添加条件`

规则列表规则：`勾上`； 情景模式：`长城防火墙`； 默认情景模式：`直接连接`。

规则列表格式：`AutoProxy`

规则列表网址：`https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`

点击更新情景模式。

------

配置完成，chrome右上角启用`自动转换`情景模式。

# 3：常用软件

## 1：Synapse

```bash
sudo pacman -S synapse
```

安装后设置激活快捷键为：`ctrl + shift + space`

## 2：BaiduPCS-Go

```bash
1：访问网址：http://pcs.baidu.com/rest/2.0/pcs/file?app_id=265486&method=list&path=%2F，生成/apps/baidu_shurufa目录。
2：config set -appid=266719
3：login -bduss=FUUUM3VU02RXlmTzRybkFXdjlkYVdaflFnSTVVNDJWSVZ-Nms1UFJaMWt2cnBjQVFBQUFBJCQAAAAAAAAAAAEAAADR4t~AS29yYW5DaGV1bmcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGQxk1xkMZNcVH
4：cd /apps/baidu_shurufa
5：config set -appid=265486
```

## 3：tim

```bash
sudo vim /etc/pacman.conf
解封multilib镜像仓库，安装64位系统中需要的32位软件和库，例如：wine等
sudo yaourt -Syu

安装tim：
sudo yaourt -Ss deepin.com.qq.office
注意查看是否安装了`deepin-fonts-wine`
```

## 4：screenfetch

# 5：桌面环境

## 1：Deepin桌面环境

更改系统音效：替换以下路径下音效文件即可

```bash
/usr/share/sounds/deepin/stereo
```

附音效文件下载网址：`http://www.aigei.com/`

# 6：安装字体

```bash
cd /usr/share/fonts
sudo mkdir myFonts
sudo mv microsoftyahei.ttf /usr/share/fonts/myFonts
sudo chmod 644 microsoftyahei.ttf
sudo mkfontscale   创建字体的fonts.scale文件,它用来控制字体旋转缩放
sudo mkfontdir     创建字体的fonts.dir文件,它用来控制字体粗斜体产生
fc-cache           扫描字体目录并生成字体信息的缓存
```