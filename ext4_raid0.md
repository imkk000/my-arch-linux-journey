# EXT4 Raid 0

## Prerequisite

- mdadm
- lvm2

## Setup Raid 0 with mdadm

- Follow this [guide](https://wiki.archlinux.org/title/RAID)
- Follow this [Raid Array](https://www.jeffgeerling.com/blog/2021/htgwa-create-raid-array-linux-mdadm)

```bash
pacman -S mdadm
mdadm --misc /dev/nvme0n1
mdadm --misc /dev/nvme1n1
mdadm --create --verbose --level=0 --raid-devices=2 /dev/md0 /dev/nvme0n1 /dev/nvme1n1
mdadm --detail --scan >> /etc/mdadm.conf
# append to this file
# ARRAY /dev/md/<NAME>:0 metadata=1.2 UUID=<UUID>
mkfs.ext4 /dev/md0
```

## Setup LVM

- Follow this [guide](https://wiki.archlinux.org/title/LVM)
- PV (Physical Volume) = Hardware
- VG (Volume Group) = Bridge
- LV (Logical Volume) = Software

```bash
pacman -S lvm2
pvcreate /dev/md0
vgcreate vgraid0 /dev/md0
lvcreate -l 100%FREE -n rootfs /dev/vgraid0
```

## Set Initial Ram Disk

```bash
pacman -S mdadm lvm2

# open /etc/mkinitcpio.conf
# add mdadm_udev and lvm2 to HOOKS
HOOKS=(... block mdadm_udev lvm2 filesystems ...)
# run generate initramfs
mkinitcpio -P
```
