---
title: Running Arch Linux with customized kernel in QEMU
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [January 31, 2019](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html "7:16 pm") 
[VOID001](https://web.archive.org/web/20210514134020/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [3 comments](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html#comments)





本文为内核爱好者们介绍一个便利的运行内核的方式，使用 QEMU + virtio 启动一个装载着自定义内核的 Arch Linux。


### 0x00 构建内核


我们使用 Arch Linux 发行版使用的 .config 作为配置文件, 可以省却很多自己配置 Kernel Options 的繁琐工作。将配置文件放入 kernel source tree 后，使用 `make oldconfig` 就可以通过一个交互的命令行将最新的内核里的可配置参数补全到我们使用的 Arch Linux 的 .config 文件中。


我们有两种方式获取到 Arch Linux (和其他发行版) 的 .config 文件:


1. 使用 zcat 读取 `/proc/config.gz` 的内容并且保存为 .config
2. 直接复制 `/usr/lib/modules/$(uname -r)/build/.config
3. 


若要使用方法 1 ，内核需要开启 `Enable access to .config through /proc/config.gz` 这一选项需要我们将 `IKCONFIG_PROC` 设置为 y （相关依赖 option 项目也需要满足）


配置文件准备好后，我们就可以开始构建内核了，如果你会频繁构建内核，建议使用 [ccache](https://web.archive.org/web/20210514134020/https://ccache.samba.org/) 来缓存编译的中间目标文件 “*.o”。ccache 具体配置方法这里不多做介绍，简单说下使用方法，很简单，只需要在 gcc,cc,g++,c++ 的前面加上 ccache 即可，e.g 使用 ccahe 编译内核的构建命令可以这样写: `make CC="ccache gcc" -j8`。注意：ccache 不会提升首次编译的速度，它会将这些中间文件缓存起来，在后续的 recompilation 中检查是否有可以命中缓存的中间文件，有的话直接使用而不用重新编译，通过这种方式提升二次编译的速度  
构建内核后，我们需要运行 make modules\_install 供 initramfs 使用相应的 kernel module。


### 0x01 构建合适的 initramfs


(建议以下操作在一个干净的 working directory 进行)


我们将会使用 QEMU 提供的 virtio device 功能来 map host OS 的 filesystem image 到 guest OS 里。因此我们需要 guest OS 支持 virtio 驱动，因为我们的 rootfs 就是 virtio device, 我们需要在内核启动的时候就加载好 virtio 驱动，这里有两个 approach: 第一个就是将该驱动作为内核的 builtin 而非 module，第二个 approach 则是在 initramfs 里加载相应的驱动。


在启动的时候，QEMU 的 bootloader 装载 kernel 以及 initramfs 到内存中，并启动内核，内核会对 initramfs 的存在进行检查，如果存在 initramfs 则将其 mount 到 / 并运行 /init (完成一系列复杂的 user-space 初始化工作) 这个过程中也会用我们在 kernel cmdline 里指定的 root disk mount 到 / 。


具体过程如下, initramfs 加载后，执行 /init 脚本, /init 中将 root disk mount 到 initramfs 的 /new\_root 上，而后通过 `switch_root` 将 mount tree 的 root 替换为 /new\_root 也就是我们在 kernel cmdline 里指定的 root disk。同时 /init 脚本也会对配置到 initramfs 里的 kernel module 进行 modprobe (insmod)，这样我们就可以通过 initramfs 启动装载合适的驱动后启动各种不同的 root disk 了( USB, RAID, dm-crypt, etc)。


因而我们需要构建合适的 initramfs 使得我们的 virtio disk 可以被加载，我们使用 Arch Linux 内的 initramfs 构建工具 mkinitcpio 进行构建。mkinitcpio 使用 .preset 文件管理构建特定 initramfs 的规则，我们编写自定义的 .preset 文件 linux-dev.preset。



```
# mkinitcpio preset file for the 'linux-macbook' package

ALL\_config="./mkinitcpio.conf"
ALL\_kver="5.0.0-rc4-macbook+"

HOOKS=()
PRESETS=('default' 'fallback')

# default\_config="/etc/mkinitcpio.conf"
default\_image="initramfs-linux-dev.img"
# default\_options=""

# fallback\_config="/etc/mkinitcpio.conf"
fallback\_image="initramfs-linux-dev-fallback.img"
fallback\_options="-S autodetect"

```

上面的 linux-dev.preset 里，我们可以只生成 default 而不生成 fallback ram image。注意 ALL\_kver 要和你编译出的内核的 kernel version 一致，不然在 ramfs modprobe 的时候会因为找不到 /lib/modules/$(uname -r)/ 导致内核在 initramfs 阶段加载我们预先定义好的驱动失败。  
为了保留系统原有的 mkinitcpio.conf 不被更改，我们复制了 mkinitcpio.conf 出来并且制定 linux-dev.preset 跟随该副本的配置, mkinitcpio.conf 我们只做一点修改，更改下加载的 MODULES 以及调用的 HOOKS :



```
MODULES="virtio virtio\_blk virtio\_pci virtio\_net ext4 xfs radeon"
HOOKS="base udev autodetect modconf block filesystems keyboard fsck"

```

大家可以根据自己的需要进行修改相应的 modules, 值得注意的是我们在 MODULES 里需要制定 virtio 系列的驱动程序，这样我们才能够让 kernel 在 bootup process 识别出我们的 virtio 设备。  
做好上述准备后，我们就可以开始生成 initramfs 了: `mkinitcpio -p ./linux-dev.preset` 。执行后我们就在当前目录获得了 initramfs-linux-dev.img 这样一个 ramdisk cpio gzip compressed image  
以下是我的 working directory 在执行完毕 initramfs 生成后的文件列表:



```
╰─(´・ω・)つ  ls -al
total 42016
drwxr-xr-x  2 void001 void001      162 Jan 31 18:03 .
drwxrwxr-x 32 void001 void001     4096 Jan 31 15:55 ..
-rw-r--r--  1 void001 void001 31195709 Jan 31 18:01 initramfs-linux-dev-fallback.img
-rw-r--r--  1 void001 void001 11802418 Jan 31 18:00 initramfs-linux-dev.img
-rw-r--r--  1 void001 void001      403 Jan 31 11:30 linux-dev.preset
-rw-r--r--  1 void001 void001     2545 Jan 31 11:24 mkinitcpio.conf
-rwxr-xr-x  1 void001 void001      428 Jan 30 19:54 mk.sh
lrwxrwxrwx  1 void001 void001       24 Jan 30 19:54 vmlinuz-linux-dev -> ../arch/x86/boot/bzImage

```

### 0x02 安装 Arch Linux 到 filesystem image


我们首先通过 qemu-img 创建 root.img home.img 两个 disk image (也可以只创建一个，根据个人喜好自行选择）  
从 [https://archlinux.org/download](https://web.archive.org/web/20210514134020/https://archlinux.org/download) 获得一个 Latest ArchISO，然后使用如下参数启动 QEMU:



```
qemu-system-x86\_64 -cdrom /path/to/your/livecd.iso --enable-kvm -m 2048 -nic user,model=virtio-net-pci # 指定 cdrom 内容为 LiveCD, 使用 KVM, 限制使用内存 2048M (过小会导致 ramfs 无法完全解压到 ram 里因而无法加载 LiveCD),使用 virtio net device 作为网络设备

# 可选参数
-nographic # 不启动图形界面 （开启该选项后可以通过当前 terminal 接管 Guest OS 的 Serial output, 需要在 cmdline 增加一个参数)
-vnc :0 # 开启 VNC Server 监听 5900 端口
```

如果我们没有添加那些可选参数那么我们可以直接通过 QEMU 的 monitor 进行 ArchLinux 的安装了，这里介绍一下这个 -nographic 额外参数。  
根据 QEMU.1(1) 说明，开启 -nographic 之后 QEMU 会将串口的输出 redirect 到 terminal(console)。为了让我们的 LiveCD 能够将内容输出到 QEMU 的 emulated Serial, 我们在 bootmenu 的 kernel cmdline 里添加: `console=ttyS0,38400` (38400 baud rate) 然后启动 LiveCD


![](https://web.archive.org/web/20210514134020im_/https://void-shana.moe/wp-content/uploads/2019/01/image-1024x293.png)修改 iso 的 bootup cmdline

> QEMU.1(1) 
> 
> -nographic  
>  Normally, if QEMU is compiled with graphical window support, it displays output such as guest graphics, guest console, and the QEMU monitor in a window. With this option, you can totally  
>  disable graphical output so that QEMU is a simple command line application. The emulated serial port is redirected on the console and muxed with the monitor (unless redirected elsewhere  
>  explicitly). Therefore, you can still use QEMU to debug a Linux kernel with a serial console. Use C-a h for help on switching between the console and monitor.


安装 Arch 的过程不需要多余的说明，如果你是第一次安装 Arch Linux 的话请 [follow the archlinux wiki](https://web.archive.org/web/20210514134020/https://wiki.archlinux.org/index.php/Installation_guide) 安装好之后，我们的 root file system image 就创建好了


### 0x03 在 QEMU 中运行 Arch Linux 并且使用自定义的内核


至此我们所有的准备工作都做好了，下面我们来启动编译好的内核，我们使用如下的 QEMU 参数



```
#!/bin/bash
BUILDROOT=/home/void001/Kernel-Hacking/latest-linux-kernel/build
KERNEL=${BUILDROOT}/vmlinuz-linux-dev
INITRAMFS=${BUILDROOT}/initramfs-linux-dev.img

qemu-system-x86\_64 -kernel ${KERNEL} \
    -initrd ${INITRAMFS} \
    -nic user,model=virtio-net-pci \
    -drive file=root.img,if=virtio,index=0 \
    -drive file=home.img,if=virtio,index=1 \
    -nographic \
    -m 2048 \
    -append "earlyprintk=ttyS0 rw root=UUID=7279a4af-7e4d-4aa0-8c19-e47da93eeb87-2333 console=ttyS0,38400 debug" \
    -vnc :0 \
    --enable-kvm

```

相比上面的 ISO bootup，这次我们指定了 kernel, initrd 两个参数，分别是对应自定义内核文件，和我们构建好的 initramfs-linux-dev.img。同时我们使用 -drive 在 guest OS 里创建两个 drive backend，interface 都是 virtio，我们已经在 initramfs 里加载了 virtio driver 因此 root filesystem 可以被正确找到并启动。-append 参数将其 value append 到 kernel cmdline 我们这里指定了 root disk (直接指定为 /dev/vd* 也是可以的，不过我这里有两个 drive 为了避免 /dev/vd* 在每次启动的时候名字可能发生变动，使用 UUID 指定了 root device)。使用上面的命令，我们就可以启动 QEMU 了，最后我们看下效果:


![](https://web.archive.org/web/20210514134020im_/https://void-shana.moe/wp-content/uploads/2019/01/image-1-1024x139.png)可以看到自定义的 kernel message 信息，这个是修改 ext4 的 module\_init 产生的。
![](https://web.archive.org/web/20210514134020im_/https://void-shana.moe/wp-content/uploads/2019/01/a-1024x538.png)Kernel is latest rc4 kernel, and CPU is using QEMU, all check
### 0x04 总结


本文介绍了一种内核爱好者可以使用的内核调试运行方法，该方法无需将内核安装到 /boot 并且进行 reboot 切换，使用 QEMU + KVM + Serial 将 guest OS 的输入输出接管到 host OS 的 terminal，方便查看，复制信息和调试，该方法也可以验证我们构建的内核是否能够在现存的 Linux Distribution 上启动成功，我们也可以将 root filesystem image mount 到 host OS 对其中的文件进行 manipulate。


### 0x05 后记


文中介绍的方法是与 @tonyluj 讨论后得到的方案，本文参考了 Arch Linux Wiki, QEMU man page 以及 tldp.org，在此加以说明。  
距离我的上一篇文章已经有很久了，这之中经历了很多事情，现在总算可以静下来继续研究内核，写文章和博客了，之后也会继续撰写 Kernel Develop / Linux 相关的文章，看了下时间也快要过春节了，最后提前祝大家春节快乐！  







---


[archlinux](https://web.archive.org/web/20210514134020/https://void-shana.moe/category/linux/archlinux), [Kernel](https://web.archive.org/web/20210514134020/https://void-shana.moe/category/kernel), [Linux](https://web.archive.org/web/20210514134020/https://void-shana.moe/category/linux) [archlinux](https://web.archive.org/web/20210514134020/https://void-shana.moe/tag/archlinux), [kernel](https://web.archive.org/web/20210514134020/https://void-shana.moe/tag/kernel), [linux](https://web.archive.org/web/20210514134020/https://void-shana.moe/tag/linux), [qemu](https://web.archive.org/web/20210514134020/https://void-shana.moe/tag/qemu) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514134020im_/https://secure.gravatar.com/avatar/3e3c1cb94b9ad3551cd9d43f2e52e105?s=50&d=identicon&r=g) **[依云](https://web.archive.org/web/20210514134020/https://blog.lilydjwg.me/)** says: 
[January 31, 2019 at 10:15 pm](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html#comment-516)
你也弄这个了呀。我当初是直接在 kvm 虚拟机里装自己打包的 linux-lily 内核来着，为了测试打的包是不是好的。
[https://blog.lilydjwg.me/2014/7/15/arch-kvm-in-arch.52548.html](https://web.archive.org/web/20210514134020/https://blog.lilydjwg.me/2014/7/15/arch-kvm-in-arch.52548.html)


	1. ![](https://web.archive.org/web/20210514134020im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[January 31, 2019 at 10:46 pm](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html#comment-518)
	ww， 原来百合喵早就弄过啦，窝这边是准备用来研究内核写代码测试用，以前都是用我的小本本直接启动，那样感觉太费劲了，然后 buildroot 的方案我还没搞明白，于是就选用了这样一个方案，目前来说感觉还是很好用的，等我什么时候搞清楚 buildroot 我再总结下他的使用方法


2. ![](https://web.archive.org/web/20210514134020im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[February 1, 2019 at 10:41 am](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html#comment-520)
David Gao 也安利我了virt-manager 我用来跑 Win10 了（


2. ![](https://web.archive.org/web/20210514134020im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
[February 1, 2019 at 10:41 am](https://web.archive.org/web/20210514134020/https://void-shana.moe/linux/running-arch-linux-with-customized-kernel-in-qemu.html#comment-520)
David Gao 也安利我了virt-manager 我用来跑 Win10 了（

            