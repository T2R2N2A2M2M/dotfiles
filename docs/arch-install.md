# Arch Linux Installation

This installation tries to dual boot Arch Linux and Microsoft Windows 11 on two separate SSD. With a few notes

- For AMD's CPU and GPU
- using `btrfs` as filesystems
- `grub` will be installed in Windows' EFI partition.
- Encrypt the root partition.
- `snapper` as our snapshots
- `zram` set up

## Initial Setup

```bash
# Larger Text
setfont ter-132b

# Verify EFI boot
cat /sys/firmware/efi/fw_platform_size

# Connect to the internet using wifi
iwctl
-> station list
-> station <station-name> get-networks
-> station <station-name> connect <wifi-name>

# Config Time
timedatectl list-timezones | grep <city_country>
timedatectl set-timezone <city_country>

# Prepare Linux drive
gdisk /dev/<linux_drive>
-> x
-> z

gdisk /dev/<linux_drive>
-> gpt
-> <linux_swap_part>:Linux Swap (e.g., 20GiB), <linux_main_part>:Linux Filesystems (e.g., The rest)
-> write
-> quit

# Encrypt disk
cryptsetup -v lukFormat /dev/<linux_main_part>
cryptsetup open /dev/<linux_main_part> main

# Setup Swap partiiton
mkswap /dev/<linux_swap_part>
swapon /dev/<linux_swap_part>

# Setup btrfs
mkfs.btrfs /dev/mapper/main
mount /dev/mapper/main /mnt

btrfs su cr /mnt/@
btrfs su cr /mnt/@home
btrfs su cr /mnt/@srv
btrfs su cr /mnt/@log
btrfs su cr /mnt/@cache
btrfs su cr /mnt/@tmp

mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@ /dev/mapper/main /mnt

mkdir -p /mnt/home
mkdir -p /mnt/srv
mkdir -p /mnt/tmp
mkdir -p /mnt/var/log
mkdir -p /mnt/var/cache

mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@home /dev/mapper/main /mnt/home
mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@srv /dev/mapper/main /mnt/srv
mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@tmp /dev/mapper/main /mnt/tmp
mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@log /dev/mapper/main /mnt/var/log
mount -o noatime,compress=zstd,commit=120,space_cache=v2,discard=async,subvol=@cache /dev/mapper/main /mnt/var/cache

# Setup boot partition
mkdir -p /mnt/boot
mount /dev/<Windows_EFI> /mnt/boot

# Mount main Windows partiiton
...
mkdir /mnt/WindowsC
mkdir /mnt/WindowsD
mkdir /mnt/WindowsE
...

...
mount /dev/<Windows_main_part_1> /mnt/WindowsC
mount /dev/<Windows_main_part_2> /mnt/WindowsD
...

# Install Arch Linux
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers xorg git wget networkmanager neovim bash-completion

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

# Zoneinfo
ls /usr/share/zoneinfo/

ln -sf /usr/share/zoneinfo/<Region/City> /etc/localtime

hwclock --systohc

# Locale
nvim /etc/locale.gen
-> English, Thai, Japanese, Korean, Chinese (Mainland, Taiwan, Hong Kong, Singapore)

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

echo "<hostname>" > /etc/hostname

# Edit pacman
nvim /etc/pacman.conf
-> Color
-> ILoveCandy
-> [multilib] & include

pacman -Syu

mkinitcpio -P

# Bootloader (grub)

pacman -S --needed btrfs-progs grub efibootmgr grub-btrfs os-prober ntfs-3g amd-ucode mtools dosfstools rsync network-manager-applet git iptables ipset man-db man-pages texinfo bluez bluez-utils pipewire alsa-utils pipewire-pulse pipewire-jack sof-firmware ttf-firacode-nerd firefox alacritty acpid reflector

blkid 
-> get UUID of <linux_main_part> 
-> echo "$(blkid -s UUID -o value /dev/<linux_main_part)"

nvim /etc/default/grub
-> comment out GRUB_DISABLE_OS_PROBER=false
-> GRUB_CMD_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=<UUID_linux_main_part>:main root=/dev/mapper/main"

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

systemctl enable fstrim.timer NetworkManager bluetooth sshd acpid reflector.timer

# AMD GPU Driver
pacman -S --needed mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader

nvim /etc/mkinitcpio.conf
-> MODULES=(btrfs amdgpu atkbd)
-> HOOKS=(... encrypt filesystems ...)

mkinitcpio -P

exit
reboot
```

## After installation

### Zram

```bash
pacman -S zram-generator

nvim /etc/systemd/zram-generator.conf
---------------------------------------
[zram0]
zram-size = ram / 4
compression-algorithm = zstd
---------------------------------------

systemctl daemon-reexec
systemctl start systemd-zram-setup@zram0.service
```

### Snapper

```bash
pacman -S snapper snap-pac

snapper -c root create-config /
systemctl enable --now snapper-timeline.timer
systemctl enable --now snapper-cleanup.timer
systemctl enable --now grub-btrfsd
```

### Permanant ter-132b font in TTY

```bash
pacman -S terminus-font

nvim /etc/vconsole.conf
-> FONT=ter-132b
```

### Add superpower user

```bash
useradd -m -g users -G wheel,storage,power <user_name>
passwd <user_name>

nvim /etc/sudoers
-> uncomment %wheel
-> add "Defaults rootpw"
```

### Hardening Suggestion
- Lock root user
- firewall (e.g., ufw)
- SSH hardening (e.g., remove passphase access, disable root login)
- Enable secure boot
- Make grub password, restrict access to grub, immutable grub, disable recovery
