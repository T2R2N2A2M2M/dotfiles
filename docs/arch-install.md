# Arch Linux Installation

This installation tries to dual boot Arch Linux and Microsoft Windows 11 on two separate SSD. With a few notes

- using `btrfs` as filesystems
- `grub` will be installed in Windows' EFI partition.
- Encrypt the root partition.
- `snapper` as our snapshots
- `zram` set up

```bash
# Larger Text
setfont ter-132b

# Verify EFI boot
cat /sys/firmware/efi/fw_platform_size

# Connect to the internet using wifi
iwctl
station list
station <station-name> get-networks
station <station-name> connect <wifi-name>

# Config Time
timedatectl list-timezones | grep <city_country>
timedatectl set-timezone <city_country>
```
