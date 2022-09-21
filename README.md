# Phytium Linux Kernel PKGBUILD for Archlinux

**This Kernel PKGBUILD for Phytium Only In Archlinux**

## 如何在一台飞腾 D2000 上安装 Archlinux ##

### 写在最前面 ###

1. 首先需要解决的就是启动，飞腾和其他的 ARM 架构系统有所不同，猜测与厂商提供的板子和 `bios` 有关，目前 Arch 上能够启动还需要些运气。按照从前的传统思维，ARM 板子要正常启动，除了 `vmlinux` （或者理解成 `vmlinuz` 也可以）， `dtb` 文件，`initramfs` 就可以了。不过推测飞腾的 `bios` 做了修改，能够直接加载 `grubaa64.efi` ，完成后续的启动。
2. 目前的 config 是从统信商业发行版那里抓来的，途径咱就不详谈了。
3. 目前这个 config 能够在 `Gentoo Linux` 和 `Archlinux` 正常启动。
4. 因为个人原因，添加了 [cjktty 补丁](https://github.com/zhmars/cjktty-patches/)和 [bbr2 补丁](https://gitlab.com/sirlucjan/kernel-patches/-/tree/master)。

### 我的板子 ###

![D2000](board/%E9%A3%9E%E8%85%BED2000.jpg)

### 目前能够成功启动飞腾 D2000 的商业发行版总结下来有优麒麟和统信，那么如何安装社区发行版就成为一个棘手的问题... ###

1. 到国内开源镜像源抓取最新版本 `ubuntu arm64 server iso`（现在最新版本是 22.04.1 ，下载[地址](https://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/22.04.1/release/ubuntu-22.04.1-live-server-arm64.iso)）；
2. 烧写到 USB 上，做成 `liveusb` 启动盘；
3. 启动机器，按 `F7` 键进入启动选项；
4. 选择刚在做好的 ubuntu arm64 liveusb ，各种回车（这个过程你的屏幕会花屏，别管它，总会出现一个选择安装的 `CLI` 界面的）；
5. 进入 `TTY2` ，就是同时按下 `Alt+Ctrl+F2` 键；
6. 输入 `sudo su` 获取 `root` 权限；
7. 使用 `passwd` 设置 `root` 密码；
8. `nano` 编辑 `/etc/ssh/sshd_config` 文件，将这个部分 `#PermitRootLogin xxx` 修改成 `PermitRootLogin yes` 保存退出；
9. 用 `systemctl restart ssh` 默认启用远程可以用 `root` 通过 `ssh` 登陆当前 `liveusb` 环境；
10. 完成远程登陆后，操作分区，挂载相关分区到目录；
11. 下载最新的 [Archlinux aarch64 rootfs 压缩包](https://mirrors.ustc.edu.cn/archlinuxarm/os/ArchLinuxARM-aarch64-latest.tar.gz)到安装路径，并通过 `tar -xpvf` 解压之；
12. `chroot` （具体命令忘记了？看下面：）
    - `cp --dereference /etc/resolv.conf /mnt/arch/etc/`
    - `mount -t proc /proc /mnt/arch/proc`
    - `mount --rbind /dev /mnt/arch/dev`
    - `mount --make-rslave /mnt/arch/dev`
    - `mount --rbind /sys /mnt/arch/sys`
    - `mount --make-rslave /mnt/arch/sys`
    - `mount --rbind /tmp /mnt/arch/tmp`
    - `mount --bind /run /mnt/arch/run`
    - `chroot /mnt/arch /bin/bash`
    - `source /etc/profile`
13. 完成后，就可以开始折腾内核了，有两个方法处理：
    - 如果你想要自己编译内核，可以直接克隆这个仓库到 chroot 环境中的 Arch 里，稍微修改下 `/etc/makepkg.conf` 中的线程数，就能通过 `makepkg -Ccs` 进行本地编译，编译提供了 menuconfig 进行配置，各位玩家可以自己稍微 DIY 下；最后编译完成通过，`pacman -U` 进行安装就可以了。**需要注意的是，默认内核用 `GCC` 编译，如果你想用 `Clang/LLVM` 编译，可以修改 `PKGBUILD` ，在 `make` 后添加 `LLVM=1 LLVM_IAS=1` 即可，编译依赖我已经写好了 clang, llvm 和 lld 了，所以不用担心会不会没有依赖。**
    - 如果你不想编译，想用我已经编译好的。将如下文本复制到 `/etc/pacman.conf` 末段：
        - `[phytium-aarch64]`
        - `SigLevel = Optional TrustAll`
        - `Server = http://hougearch.litterhougelangley.club/aarch64/`
    - 添加完成后，运行 `pacman -Sy; pacman -S linux-phytium linux-phytium-headers` 即可安装内核及对应头文件。
14. 内核添加了 cjktty 和 bbr2 补丁，方便国人使用。
15. 完成安装后，别着急重启，需要稍微调整下 `/boot` 目录下的内容，里面默认只有 `Image` `Image.gz` `initramfs` 等这类的文件，还需要额外通过 `pacman` 安装如下几个包：`grub` `dosfstools` `efibootmgr`，然后将 `Image.gz` 复制成 `vmlinuz-linux`，随后可以用 `mv` 命令把 `Image` 和 `Image.gz` 改成其它名字，目的是避免 grub-mkconfig 时出现多个内核。完成后，该目录下应该有 `initramfs-xxx.img` 和 `vmlinuz-xxx` 就可以了，再利用 `gurb` 制作 `bootloader`，具体操作如下：
    - `grub-install --efi-directory=/boot --bootloader-id=GRUB`
    - `grub-mkconfig -o /boot/grub/grub.cfg`
16. （如果重启后还是不能启动）可以尝试 `grub-install --efi-directory=/boot --bootloader-id=Ubuntu` 或 `grub-install --efi-directory=/boot --bootloader-id=Deepin` 等神奇的魔法。
17. 最后别忘记配置 `root` 密码，是否需要 `Archlinux` 允许远程 `ssh root` 登陆等相关细节事宜就可重启完成安装了。
18. 大家不要忘记默认 Archlinux ARM 是有默认普通用户的，叫做 `alarm` （默认登陆密码是 alarm，拥有使用 sudo 的权限），如果不需要就删掉。

## 显卡问题 ##

1. 目前最佳实践是使用 AMD 显卡，群友大部分都使用 rx550 测试，一般情况来说，AMD 现在最有把握。
2. 如果使用 NVIDIA 显卡，就需要分两种情况来说了。
    - 使用开源驱动，nouveau 可以驱动开普勒架构及其以前的型号；nvidia-open 驱动目前没有打包进入 Archlinux，不清楚后续是否有计划。
    - 也可以使用我打包好的 nvidia-aarch64 的包，同样在上面那个源可以下载使用。
