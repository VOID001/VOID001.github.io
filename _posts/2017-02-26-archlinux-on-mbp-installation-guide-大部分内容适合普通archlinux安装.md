---
title: ArchLinux on MBP Installation Guide [大部分内容适合普通ArchLinux安装] 上
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [February 26, 2017](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html "12:49 am") 
[VOID001](https://web.archive.org/web/20210614024104/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [10 comments](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comments)





TL;DR
-----


这两天OS X的虚拟机挂了，两个月的工作成果都丢了，因为本人是重度Archlinux依赖用户既然虚拟机这么难用（不能用全部内存&有快照损坏的风险）因而，唯一的选择就是在MBP上装Archlinux了。同时因为这是第一次用UEFI的模式Dual boot Archlinux + OS X，并且是第一次使用KDE而不是一直用的Gnome,因而踩了很多坑，所以本文会比较长（


本文介绍的是在MBP上Dualboot OS X + ArchLinux 使用KDE并配置好所需的必备软件的整个过程


ArchLinux 优点
------------


如果连这个都不知道的话那么说明可能温豆师/污班图还是比较适合你（（其实是很麻烦不想写因为已经写烂大街了


那么我们就开始吧OwO


材料准备
----


* 一个容量足够装下ArchISO的U盘
* 一台Macbook
* 畅通的网络
* 一些干粮（不然熬久了会饿的）
* rEFInd
* 不确定是否要禁用Apple的 [Configuring System Integrity Protection](https://web.archive.org/web/20210614024104/https://developer.apple.com/library/content/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html)如果在操作引导过程中遇到问题，那么就设置一下这个
* 足够大的空间（用OS X 自带的 Disk Util 将要用来装Arch的空间划分出来，不用管文件系统，反正一会儿也得删）


引导安装
----


### EFI 介绍


EFI boot是比 BIOS Boot 要先进的 boot 方式，古老的 BIOS 需要让CPU先进入16bit的实模式，仅仅能执行有限的一些 BIOS 提供的中断，并且 BIOS因为是直接用CPU的汇编编写的，对硬件平台有非常高的依赖，包括在BIOS下运行的驱动，尤其网络驱动，还要独立的给每一个架构的CPU编写一套独立的驱动，维护难度和开发难度都比较高，而 EFI 加载的驱动是以 EFI 字节编码 ，独立于CPU架构，因而更优，且BIOS无法支持大于 2TiB 的硬盘，因而对目前的很多机器这都是致命的瓶颈 （[参考资料](https://web.archive.org/web/20210614024104/http://www.tldp.org/HOWTO/Large-Disk-HOWTO-4.html)）， 因而目前的个人PC开始采用 UEFI （EFI的一个更新版本）进行系统的引导。


### 引导过程以及ESP


EFI 的引导不同与 BIOS，不需要将 Bootable program 放在 first sector ，而是将引导程序放在 ESP 中


ESP (EFI System Partition) 存放了EFI引导的必备程序，路径格式需要符合此规范<EFI\_SYSTEM\_PARTITION>/BOOT/BOOT<MACHINE\_TYPE\_SHORT\_NAME>.EFI （此行摘自wikipedia）


例如/efi/BOOT/BOOT （具体识别哪个路径，以及能否识别不同的路径跟固件的实现有关，具体讨论如下


 



```
VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:20]
继续提问喵， ESP窝看的文档资料都说要有一个EFI/BOOT/BOOT<ARCH>.efi这种结构的目录才可以检测到，为什么窝的目录是EFI/EFI/refind/refind\_x64.efi他也能识别呢

ヨイツの賢狼ホロ | ?, [24.02.17 16:21]
?有些坑爹系统只能识别 EFI/BOOT/BOOT<ARCH>.efi

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:22]
就是说现在新的firmware甚至可以识别只要是GUID = EFI\_GUID 并且 fs=FAT的

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:22]
.efi文件存在的都能引导？

farseerfc ?, [24.02.17 16:22]
[In reply to VOID 001 | 欢迎加入 #dev-horo =w=]
看 UEFI 實現，有的 UEFI 只認 EFI/BOOT/BOOT<ARCH>.efi ，有的 UEFI 就可以 efivars 裏面設

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:23]
这个不是软件层的实现吧？

farseerfc ?, [24.02.17 16:23]
我的 vaio 的 UEFI 裝了 win10 之後連 EFI/BOOT/BOOT<ARCH>.efi 都不認

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:23]
这个是固件吧？

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:23]
除了刷固件没法改是吧0w0

farseerfc ?, [24.02.17 16:23]
嗯是固件

VOID 001 | 欢迎加入 #dev-horo =w=, [24.02.17 16:24]
[In reply to farseerfc ?]
（那它认什么OO

farseerfc ?, [24.02.17 16:24]
[In reply to VOID 001 | 欢迎加入 #dev-horo =w=]
認 EFI/Microsoft/Windows/bootmgfw.efi ，所以我把這個文件的內容替換成 systemd-boot.efi 的內容了（
```

）


识别之后，就会加载相应的程序以及驱动，之后就是.efi程序接管启动过程了


### 操作


这里采用的是使用rEFInd替换OS X原生的引导。


首先呢我们要将引导准备好, 以下操作在OS X下进行


在 [rEFInd](https://web.archive.org/web/20210614024104/https://sourceforge.net/projects/refind/) 下载安装程序, 安装好rEFInd


### 


http://www.rodsbooks.com/refind/installing.html 这里的教程应该是有点过时了，安装 refind 现在可以简单的直接运行`refind-install指令`


refind-install 会自动识别一般的Mac的 EFI 分区并且将refind<arch>.efi复制进去


然后需要修改OS X的默认boot loader，这里使用 OS X 提供的 bless 工具即可  （如果无法写入的话，关闭System Integrety Protection) 


sudo bless –mount /Volumes/ESP –setBoot –file /Volumes/ESP/efi/refind/refind\_x64.efi –shortform


然后关机，再次开机之后应该就能看到rEFInd的界面啦～


![](https://web.archive.org/web/20210614024104im_/https://voidisprogramer.com/wp-content/uploads/2017/02/screenshot_001-1024x640.png)


下一步就是开始进行ArchLinux的安装辣～ 请各位小伙伴准备好ArchLinux的wiki，查看Installation Guide与Macbook这两个词条哦～


基本安装
----


首先制作好LiveCD，LiveCD的制作和在其他电脑上安装 Archlinux 没有区别，然后重启Boot到ArchLinux LiveCD中即可


然后下面开始ArchLinux安装的基本步骤:


***调网络,时间=>分区=>格式化=>挂盘=>装底包=>chroot=>各种基本配置=>装引导=>重启***


为了照顾到安装还不熟练的小伙伴们，这里会把每个过程都列出来（其实我也不熟练，也就装了十几次（逃


### 网络和时间设置


考虑到Macbook大多数没有有线以太网口，这里以无线网链接为例来进行介绍，我们使用wpa\_supplicant进行处理。


首先检测自己的网络设备


ip link list  找到自己的无线设备，然后将设备设置为up状态


ip link set <dev> up


然后使用wpa\_supplicant链接无线网络，具体用法如下


wpa\_supplicant -Dnl80211,wext -i <dev> -c<(wpa\_passphrase “YourNetWorkSSID” “YourNetworkPass”)  这里解释一下该指令的含义，wpa\_supplicant 是用于进行WiFi认证的客户端，指定的参数为：驱动使用 nl80211 或者 fallback 为 wext 驱动（前者为新的netlink interface驱动），使用的网络接口设备为 <dev> ，链接用的配置文件从重定向读入，重定向是一个shell指令，wpa\_passphrase用来生成可以被wpa\_supplicant读取的配置文件。


测试如果可以链接之后，Ctrl+C 停掉进程，加上-B参数以后台模式运行wpa\_supplicant。


下面获取IP，推荐WiFi开启dhcp服务，之后直接通过 linux 的 dhcpcd client 获取 IP 即可



```
dhcpcd <dev>
```

之后就可以尝试


ping archlinuxcn.org 辣


### 分区


窝们采用 UEFI 在MBP上 Dual Boot OS X & ArchLinux ，因而分区的创建要注意几点


* 第一个Archlinux的分区要和OS X分区有128MB的空隙，不然会导致OS X无法正常使用
* 设置好 UEFI 分区的GUID


分区大家可以使用自己喜欢的工具进行，这里要注意的就是，给Arch分第一个分区的时候要在First Sector中填入+128M(有的工具不支持，窝只在cgdisk里用过) ,然后注意EFI分区大小要大于200MB，并且设好GUID，其他分区自行判断辣～


这里给出本人的一个分区方案



```
sda      8:0    0 465.9G  0 disk 
├─sda1   8:1    0   200M  0 part /boot/efi
├─sda2   8:2    0    28G  0 part 
├─sda3   8:3    0 619.9M  0 part 
├─sda4   8:4    0   200M  0 part /boot
├─sda5   8:5    0    80G  0 part /
├─sda6   8:6    0     4G  0 part [SWAP]
├─sda7   8:7    0    20G  0 part /var
└─sda8   8:8    0 332.8G  0 part /home

```

### 格式化


窝采用了 xfs 作为文件系统， 过一阵子考虑做一个对不同文件系统的对比，介绍的文章（坑）。因而将/home /var / 全部格式化为 xfs格式，把  /boot 格式化为 vfat 格式， 这里给出格式化/boot分区的指令


mkfs.fat -F32 /dev/sdxY


格式化好之后就可以挂盘开始装基础包辣～


### 挂载硬盘（/boot的挂载是重点）


这里要注意，窝们使用的是 UEFI 的方式 boot 系统，那么要配置好 ESP (EFI System Partition) 使之符合 ESP 的文件目录规范～不过这里说是规范呢，实际上是和具体实现有关的，好在Mac上的（至少窝的macbook pro 15）firmware会遍历/efi下的*.efi文件，找到了就会以此作为boot loader 或者 bootable device (具体寻找顺序，未知，待实验）


OS X已经有现成的 ESP 了， 我们不妨就直接拿来用， 将 apple 的 ESP （在我电脑里是 /dev/sda1)挂载到 /boot/efi 上，在此之前先挂载好 /boot ，其他盘的挂载方式都按照wiki来即可，挂载在 /mnt 下，顺序为先挂 / 的分区，然后挂 /home /var /boot 然后在 /boot 里创建 efi 挂载点，把 esp 挂载到 efi 挂载点上，挂载之后的样子应该和窝上面给出的格式类似。



```
mount /dev/sda5 /mnt
mount /dev/sda7 /mnt/var
mount /dev/sda8 /mnt/home
mount /dev/sda4 /mnt/boot
mkdir -p -v /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

挂载好之后，并且确定网络正常，就可以开始正式的安装了～


### 装底包


在安装前先选择好速度较快的镜像， 修改 /etc/pacman.d/mirrolist 将想要使用的镜像放到最前面，即可


然后就可以 pacstrap /mnt base 了～ 这个过程结束之后， archlinux 的基本包就装好了～


### chroot & 基本配置


在chroot之前，我们来建立一下开机必备的fstab文件。


### fstab 介绍


fstab 是文件系统的静态信息（翻译自man page) 简单理解这个信息就是系统启动的时候对磁盘分区的挂载的指示，如果没有此信息或者此信息存在错误，那么很可能因为磁盘挂载失败导致开机失败。


以下是一个 fstab entry 的示例



```
LABEL=t-home2   /home      ext4    defaults,auto\_da\_alloc  0  2
```

用空格和tab将一个entry分为了多个部分:


* 第一部分是 用于标示分区的部分，可以是分区名(/dev/sda2)，也可以是分区 UUID (UUID=xxxx)，或者label，或者GPT分区的UUID,Label 不过强烈不建议使用分区名作为分区的标示依据，分区名会根据连接的设备不同而变化，比如目前是/dev/sda的硬盘可能在你连了一个移动硬盘之后就变成/dev/sdb了，这样以来fstab就全乱了
* 第二部分是挂载点
* 第三部分是文件系统类型
* 第四部分是mount时候的选项，比如 ro 只读挂载，还有特定文件系统自带的一些选项
* 第五部分（未知）
* 第六部分是mount的顺序，编号越小越先挂载


好了，介绍完fstab了，那么现在我们就来生成fstab，注意一定要在生成的时候加上 -U 选项不然的话生成的就不是以UUID标示分区的fstab，而是以分区设备名标识的了（惨痛的教训



```
genfstab -U /mnt > /mnt/etc/fstab
```

然后接下来就可以chroot了～



```
arch-chroot /mnt
```

之后就是一些必要的设置，以及引导的安装啦


首先我们设置一下时区, 并生成 /etc/adjtime



```
 ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
 hwclock --systohc
```

之后设置必要的语言环境(locale)，如果设置语言环境出了问题，就会出现各种神奇的乱码，让你欲罢不能（很烦


国内的用户需要用到的locale应该至少有 en\_US.UTF-8 zh\_CN.UTF-8 这两个，所以我们把这两行从 /etc/locale.gen 中解除注释 然后生成 locale



```
locale-gen
```

在 /etc/locale.conf 中设置好默认语言LANG



```
LANG=en\_US.UTF-8
```

然后建立一个 /etc/hostname 文件，里面给自己心爱的 Arch 酱起个名字吧～～



```
echo "ShakuganNoArch" >> /etc/hostname
```

不要忘了给root用户设置一个密码，下一次登录要使用



```
passwd
```

好啦，经过这些设置之后，我们已经有一个基本功能的 Archlinux 了，下面我们要 Boot 到安装好的 ArchLinux 中，离开 LiveCD，在这之前，我们还有一些事情要做~


 


### 安装引导程序


refind 需要一个config file来 boot 我们的ArchLinux，这个configfile可以通过 refind-install 生成一个框架，然后我们需要修改一下这几个entry，因为默认生成的 entry 不能用（（（


使用refind-install之后，我们应该能在 /boot/ 下看到一个文件 refind\_linux.conf ，没错就是他，而且他应该和vmlinuz-linux以及initramfs在一个分区下才正确 。


我们打开这个文件，修改它的内容如下～



```
"ShakuganNoArch"   "rw root=UUID=aacfc516-ff85-4087-84b9-8a269b88dde8 add\_efi\_memmap acpi\_backlight=video linux=/boot/vmlinuz-linux initrd=\initramfs-linux.img"
"ShakuganNoArch -- Fallback initrd"   "rw root=UUID=aacfc516-ff85-4087-84b9-8a269b88dde8 add\_efi\_memmap initrd=/boot/initramfs-linux.img"
"ShakuganNoArch -- Terminal Only"   "rw root=UUID=aacfc516-ff85-4087-84b9-8a269b88dde8 add\_efi\_memmap systemd.unit=multi-user.target"

```

恩恩～～ 那么只要重启，就能（应该）看到我们的Arch Linux辣~ 我们将会在真正的ArchLinux而不是ArchISO里进行后续的图形界面设置，以及自定义设置


 


不过在此之前，为了下一步能够进行下去，我们需要安装无线网络工具，比如 netctl 或者 刚刚使用的 wpa\_supplicant， 这两个包是没有随着 base 一起装进来的。


 


那么我们这次就说到这里～ 下一篇文章将会介绍详细的设置，以及 MBP 的几个常见使用问题的解决办法






---


[archlinux](https://web.archive.org/web/20210614024104/https://void-shana.moe/category/linux/archlinux), [Linux](https://web.archive.org/web/20210614024104/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210614024104/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/6b014a89bdd611ea6bc6465c36478f09?s=50&d=identicon&r=g) **[nian](https://web.archive.org/web/20210614024104/http://whoisnian.com/)** says: 
[February 26, 2017 at 1:08 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-238)
有点心动了


	1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[February 26, 2017 at 11:05 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-239)
	=w=不妨来试试


2. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/1c3c8c31f2744aab520428aa713c66e4?s=50&d=identicon&r=g) **[Sherlock Holo](https://web.archive.org/web/20210614024104/https://sherlock-holo.github.io/)** says: 
[March 1, 2017 at 1:27 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-240)
fstab是告诉内核兄如何挂载文件系统，按照wiki的说法是filesystem table，直译过来就是文件系统表吧


	1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/1c3c8c31f2744aab520428aa713c66e4?s=50&d=identicon&r=g) **[Sherlock Holo](https://web.archive.org/web/20210614024104/https://sherlock-holo.github.io/)** says: 
	[March 1, 2017 at 1:28 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-241)
	orz这头像。。。


	2. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[March 1, 2017 at 10:06 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-242)
	OwO 是的，我看了一下man page说的是fstab是文件系统的静态信息，这个不是他的名字，应该只是他的用处的描述


3. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[March 9, 2017 at 9:38 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-244)
已经安利辣~


4. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/731a1b7967f19dca4a6f4886d4082354?s=50&d=identicon&r=g) **天才琪露诺** says: 
[May 30, 2017 at 11:25 pm](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-274)
哇原来home要分配这么大空间的。。。看了下发现好像已经要不够用了。。。


	1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[May 31, 2017 at 10:07 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-275)
	OwO是呀因为自己的东西都存在home嘛～


5. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/cb1e7aaacaa7a0e693c864133eb8da44?s=50&d=identicon&r=g) **akka** says: 
[July 29, 2018 at 11:58 pm](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-430)
关于boot部分真没看懂，写的不清不楚不知所以嘛


	1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[July 30, 2018 at 11:47 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-431)
	感谢建议, 可否具体说明哪里没有写清楚, 便于我修改本文使得内容更准确和实用呢?


	1. ![](https://web.archive.org/web/20210614024104im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[July 30, 2018 at 11:47 am](https://web.archive.org/web/20210614024104/https://void-shana.moe/linux/archlinux-on-mbp-installation-guide-%e5%a4%a7%e9%83%a8%e5%88%86%e5%86%85%e5%ae%b9%e9%80%82%e5%90%88%e6%99%ae%e9%80%9aarchlinux%e5%ae%89%e8%a3%85.html#comment-431)
	感谢建议, 可否具体说明哪里没有写清楚, 便于我修改本文使得内容更准确和实用呢?

            