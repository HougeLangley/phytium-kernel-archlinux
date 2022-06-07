# Phytium Linux Kernel PKGBUILD for Archlinux

**This Kernel PKGBUILD for Phytium Only In Archlinux**

## 如何在一台飞腾 D2000 上安装 Archlinux ##

### 写在最前面 ###

1. 首先需要解决的就是启动，飞腾和其他的 ARM 架构系统有所不同，猜测 `dts` 和主板配合后，不需要特别在意 `dtb` 文件，只需要一个正确的飞腾 D2000 内核配置文件，即 `config` ，`grub`，能够正常生常 `initramfs` 基本这个系统就能起来了。
2. 目前的 config 是从统信商业发行版那里抓来的，途径咱就不详谈了。
3. 目前这个 config 能够在 `Gentoo Linux` 和 `Archlinux` 正常启动。
4. 如何

### 目前能够成功启动飞腾 D2000 的非商业发行版总结下来有优麒麟和统信，那么如何安装社区发行版就成为一个棘手的问题... ###

1. 到国内开源镜像源抓取最新版本 `ubuntu arm64 server iso`（现在最新版本是 22.04）；
2. 烧写到 USB 上，做成 `liveusb` 启动盘；
3. 启动机器，按 `F7` 键进入启动选项；
4. 选择刚在做好的 ubuntu arm64 liveusb ，各种回车（这个过程你的屏幕会花屏，别管它，总会出现一个选择安装的 `CLI` 界面的）；
5. 进入 `TTY2` ，就是同时按下 `Alt+Ctrl+F2` 键；
6. 输入 `sudo su` 获取 `root` 权限；
7. 使用 `passwd` 设置 `root` 密码；
8. `nano` 编辑 `/etc/ssh/sshd_config` 文件，将这个部分 `#PermitRootLogin xxx` 修改成 `PermitRootLogin yes` 保存退出；
9. 用 `systemctl enable --now ssh` 默认启用远程可以用 `root` 通过 `ssh` 登陆当前 `liveusb` 环境；
10. 完成远程登陆后，常规操作分区，挂载相关分区到目录；
11. 下载最新的 Archlinux aarch64 rootfs 压缩包到安装路径，并通过 `tar -xpvf` 解压之；
12. `chroot` （具体命令忘记了？看下面：）
    - `cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`
    - `mount -t proc /proc /mnt/xxx/proc`
    - `mount --rbind /dev /mnt/xxx/dev`
    - `mount --make-rslave /mnt/xxx/dev`
    - `mount --rbind /sys /mnt/xxx/sys`
    - `mount --make-rslave /mnt/xxx/sys`
    - `mount --rbind /tmp /mnt/xxx/tmp`
    - `mount --bind /run /mnt/xxx/run`
    - `chroot /mnt/xxx /bin/bash`
    - `source /etc/profile`
13. 完成后，将内核下载，并通过 `pacman -U` 安装；
14. 调整下 `/boot` 目录下的内容，里面默认只有 `Image` `Image.gz` `initramfs` 等这类的文件，需要安装如下几个包：`grub` `dosfstools` `efibootmgr`，然后将 `Image` 或者 `Image.gz` 复制成 `vmlinuz-linux`，这样目录下就有 `vmlinuz` 和 `initramfs` 了，再利用 `gurb` 制作 `bootloader`，具体操作如下：
    - `grub-install --efi-directory=/boot --bootloader-id=GRUB`
    - `grub-mkconfig -o /boot/grub/grub.cfg`
15. 最后别忘记这是 `root` 密码，配置好 `Archlinux` 是否允许远程 `ssh root` 登陆等相关细节事宜就可重启完成安装了。
16. 大家不要忘记默认 Archlinux ARM 是有默认普通用户的，叫做 `alarm` ，如果不需要就删掉。
