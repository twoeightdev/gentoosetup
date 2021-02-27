#### Simple gentoo installation guide for my machine

- wipefs -a /dev/sdx

### Partition

- parted -a optimal /dev/sdb
- mklabel gpt
- unit mib
- mkpart primary 1 3
- name 1 grub
- set 1 bios_grub on
- mkpart primary 3 131
- name 2 boot
- mkpart primary 131 4227
- name 3 swap
- mkpart primary 4227 -1
- name 4 rootfs
- print and check
- write
- quit
- mkfs.fat -F 32 /dev/sdb2
- mkfs.ext4 /dev/sdb4
- mkswap /dev/sdb3
- swapon /dev/sdb3

### Mount
- mount /dev/sdb4 /mnt/gentoo

### Install Stage 3
- date
- ntpdate -u pool.ntp.org
- cd /mnt/gentoo
- links https://www.gentoo.org/downloads/
- tar xpvf stage3-* --xattrs-include='*.*' --numeric-owner
- rm -rf stage3-*

### Configure compile options
- nano -w /mnt/gentoo/etc/portage/make.conf
- copy make.conf
- mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
- mkdir --parents /mnt/gentoo/etc/portage/repos.conf
- cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
- change sync-rsync-verify-metamanifest = yes to no
- cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

### Mount necessary filesystems
- mount --types proc /proc /mnt/gentoo/proc
- mount --rbind /sys /mnt/gentoo/sys
- mount --rbind /dev /mnt/gentoo/dev
- mount --make-rslave /mnt/gentoo/sys
- mount --make-rslave /mnt/gentoo/dev

### Chroot
- chroot /mnt/gentoo /bin/bash
- source /etc/profile
- export PS1="(chroot) ${PS1}"

### Mount Boot
- mount /dev/sdb2 /boot
- emerge-webrsync

### Setup portage
- cd /etc/portage
- mkdir package.accept_keywords
- echo "app-editors/neovim nvimpager tui luajit" > package.use/neovim

### Updating the @world set
- emerge --verbose --update --deep --newuse @world
- emerge -q app-editors/neovim

### Timezone
- echo "Asia/Manila" > /etc/timezone
- emerge --config sys-libs/timezone-data
- cp /usr/share/zoneinfo/Asia/Manila /etc/localtime

### Locales
- nvim /etc/locale.gen
- add en_PH.UTF-8 UTF-8
- locale-gen
- eselect locale list
- eselect locale set X = select en_PH.UTF-8
- env-update && source /etc/profile && export PS1="(chroot) ${PS1}"

### Configure kernel
- emerge -q sys-kernel/gentoo-sources genkernel
- emerge -q sys-apps/pciutils
- emerge -q app-arch/lzop app-arch/lz4
- cd /usr/src/linux
- make menuconfig

- Processor type and features to disable
  - AMD MCE features
  - AMD microcode loading support

- General setup to change
  - Kernel compression mode select LZ4
- to disable
  - Support initial ramdisk/ramfs compressed using gzip, bzip2, LZMA, XZ, LZ0, ZSTD

- Virtualization to enable
  - Kernel-base Virtual Machine (KVM) support
    - KVM for Intel (and compatible) processors support

- Device drivers to disable
  - Macintosh device drivers

- File systems to enable
  - FUSE (Filesystem in Userspace) support
  - Virtio Filesystem

- Audio = Device Drivers/Sound card support/Advance Linux Sound Architecture/HD Audio
  - Build Analog Devices HD-audio codec support - enable this

- Mic = Device Drivers/Sound card support/Advance Linux Sound Architecture/USB sound devices
  - USB Audio/MIDI driver - enable this

- Save and exit
- make && make modules_install && make install
- genkernel --install --kernel-config=/usr/src/linux/.config initramfs
- check if it exists with ls /boot/initramfs*


- set hostname in nvim etc/conf.d/hostname
```
hostname="gentoo"
```
- emerge --ask --noreplace net-misc/netifrc
- nvim /etc/conf.d/net
- set to config_enp0s31f6="dhcp"
- emerge net-misc/dhcpcd
- cd /etc/init.d
- ln -s net.lo net.enp0s31f6
- rc-update add net.enp0s31f6 default
- vim /etc/hosts
```
127.0.0.1 gentoo.homenetwork gentoo localhost
::1       gentoo.homenetwork gentoo localhost
```

### Fstab and Bootloader
- get UUID blkid -s UUID -o value /dev/sdb2
- blkid -s UUID -o value /dev/sdb3
- blkid -s UUID -o value /dev/sdb4
- emerge --verbose sys-boot/grub:2
- grub-install --target=x86_64-efi --efi-directory=/boot
- grub-mkconfig -o /boot/grub/grub.cfg
- nvim /etc/fstab
```
/boot/efi    vfat    defaults    0 2
/none swap sw 0 0
/    ext4    noatime    0 1
```
- emerge -q app-admin/sudo
- nvim /etc/sudoers - uncomment wheel

### Packages
- emerge -q package
- x11-libs/libX11
- x11-base/xorg-drivers
- x11-base/xorg-server
- x11-libs/libXrandr
- x11-libs/libXinerama
- x11-libs/libXft
- x11-apps/xinit
- x11-apps/xrdb
- x11-apps/mesa-progs
- x11-apps/xrandr
- x11-drivers/xf86-video-vboxvideo
- x11-drivers/nvidia-drivers
- x11-misc/unclutter
- x11-misc/xclip
- x11-misc/pcmanfm
- x11-terms/xterm
- x11-terms/kitty
- sys-process/htop
- media-sound/alsa-utils
- www-client/firefox
- media-fonts/fontawesome
- media-fonts/libertine
- media-fonts/inconsolata
- dev-libs/libltdl
- media-libs/libv4l
- dev-vcs/git

- rc-config add elogind default
- rc-service elogind start

### Add user
- passwd
- useradd -m -G users,wheel,video,audio -s /bin/bash jl
- passwd jl
- exit
- cd
- umount -R /mnt/gentoo
- reboot

### Adjust volume ALSA
- amixer -D pulse sset Master 5%+
- amixer -D pulse sset Master 5%-

### Adjust volume PULSEAUDIO
- pactl set-sink-volume @DEFAULT_SINK@ +10%
- pactl set-sink-volume @DEFAULT_SINK@ -10%
- pactl set-sink-mute @DEFAULT_SINK@ toggle - to mute by toggle






