# For Future Me

## Why?

- READ CAREFULLY [Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- This is my first time installing Arch Linux. I want to learn new experiences and have a better understanding of the system.
- I do not use `archinstall`. I want to do it manually.
- I did many mistakes for the first time. I didn't read installation guide carefully. After spend a lot of time, I finally got it working. I hope this guide will help future me to avoid the mistakes I made.

## Connect to wifi

```bash
iwctl
> station wlan0 connect <SSID>
```

## Update pacman

```bash
pacman -Sy

# Update keys for old live installation
pacman -Sy archlinux-keyring
```

## Setup btrfs (reduce mdadm+lvm2)

```bash
mkfs.btrfs -L raid0 -d single -m raid0 /dev/nvme0n1 /dev/nvme1n1

# mount to create new subvolumes
mount /dev/nvme0n1 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
umount /mnt

# mount subvolumes as directories
mount -o subvol=@ /dev/nvme0n1 /mnt
mount -o subvol=@home --mkdir /dev/nvme0n1 /mnt/home

# list subvolumes
btrfs subvolumes list -t /mnt
```

## Create partitions

- Use SATA SSD for boot and swap

```bash
fdisk /dev/nvme2n1
> g
> n
> +1G (for bootloader)
> t uefi
> n
> +8G (for ram 64GB)
> t
> swap
> w (write)
```

## Format partitions

```bash
# rootfs
mkfs.ext4 /dev/vgraid0/rootfs

# efi and swap
mkfs.fat -F 32 /dev/efi_system_partition
mkswap /dev/swap_partition
```

## Mount Partitions

```bash
mount /dev/vgraid0/rootfs /mnt
mount --mkdir /dev/efi_system_partition /mnt/boot
swapon /dev/swap_partition
```

## Install Base System

```bash
pacstrap -K /mnt base linux linux-firmware

# Generate /etc/fstab
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot into the new system

```bash
arch-chroot /mnt
```

## Set Details

```bash
ln -sf /usr/share/zoneinfo/Asia/Bangkok /etc/localtime

sudo nano /etc/locale.gen
# Uncomment: en_US.UTF-8 UTF-8
locale-gen

cat <<EOF > /etc/locale.conf
LANG=en_US.UTF-8
EOF

cat <<EOF /etc/hostname
VM-PC
EOF

passwd root

useradd -mU kk -s /usr/bin/fish
passwd kk

# visudo NOPASSWD (TOO Risky, But Lazy)
kk ALL=(ALL:ALL) NOPASSWD: ALL
```

## Install Pipewire (Sound)

```bash
pacman -S pipewire-jack pipewire pipewire-pulse
```

## Install Bluez (Bluetooth)

```bash
# pulseaudio-bluetooth (optional)
pacman -S bluez bluez-utils bluedevil
systemctl enable bluetooth.service
```

## Mount External Disks

```bash
lsblk -f

# add into /etc/fstab
# UUID=b8e4a1f6-6e8f-4f6e-8b6e-6e8f4f6e8b6e /mnt/partition ext4 defaults 0 0
UUID=dda55b67-9c78-4798-bb29-beeb7da6f2f6 /mnt/BACKUP_8TB ext4 defaults 0 0
UUID=b474193d-cd24-4258-b735-bea46c595733 /mnt/BACKUP_2TB ext4 defaults 0 0
```

## Install Grub

```bash
pacman -S grub efibootmgr

# https://wiki.archlinux.org/title/GRUB
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

## Install required tools

```bash
pacman -S fish git sudo openssh
pacman -S man xclip tmux neovim tree htop
pacman -S ripgrep eza bat zoxide
pacman -S btrfs-progs
pacman -S go
pacman -S otf-monaspace
pacman -S diff-so-fancy luarocks yarn
pacman -S torsocks ufw
pacman -S unrar unzip
pacman -S rsync
pacman -S nix

# custom archlinux iso
pacman -S archiso

yarn global add npm
```

## Install default software

```bash
yay -S ghostty
yay -S visual-studio-code-bin obsidian
yay -S vesktop-bin postman-bin
yay -S claude-code claude-desktop-bin
yay -S feishin-bin
yay -S librewolf-bin brave-bin
yay -S 1password
yay -S cloudflared

# optional
yay -S firefox google-chrome waterfox-bin

# tor onion
gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org
yay -S tor-browser-bin

yay -S ollama
systemctl enable ollama
```

## Install Nvidia

- Current driver: `nouveau`
- [Nvidia](https://wiki.archlinux.org/title/NVIDIA)
- [CodeNames](https://nouveau.freedesktop.org/CodeNames.html)
- [OpenRGB](https://archlinux.org/packages/extra/x86_64/openrgb/)

```bash
# peek in pci
lspci -k -d ::03xx

# for 3060 (NV170 - Ampere)
pacman -S nvidia-open
```

## Install Network Manager

```bash
# plasma-nm (Network Manager UI)
pacman -S networkmanager plasma-nm

systemctl enable NetworkManager
```

## Install Desktop Manager

- [Desktop Environments](https://wiki.archlinux.org/title/Desktop_environment)
- [SSDM](https://wiki.archlinux.org/title/SDDM)
- [KDE](https://wiki.archlinux.org/title/KDE)

```bash
pacman -S plasma-desktop sddm
pacman -S dolphin spectacle kscreen ktorrent kwalletmanager
pacman -S gwenview ark

# optional
pacman -S kde-applications
systemctl enable sddm
```

## Install YAY (AUR Helper)

```bash
# https://github.com/Jguer/yay
sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
```

## Podman (Rootless)

```bash
pacman -So podman podman-compose podman-docker
systemctl enable podman.service

mkdir -p ~/.config/containers/systemd/
systemctl --user daemon-reload

# always run unless no login
loginctl enable-linger $USER
```

## UFW

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose

sudo ufw allow ssh
```

## Hidraw

```bash
# on
sudo chmod a+rw /dev/hidraw*

# off
sudo chmod 0600 /dev/hidraw*
```

## Install Virtualbox

```bash
yay -S virtualbox virtualbox-host-modules-arch
```

## Install VMWare

```bash
yay -S uefitool-bin linux-headers
yay -S vmware-workstation
# install after workstation
yay -S vmware-keymaps

# disable secure boot
sudo modprobe -a vmw_vmci vmmon

# enable network service
systemctl enable vmware-networks.service
```

## Install LLM

```bash
yay -S cuda anaconda llama.cpp
```

## Install Dig

- Bind is dns server package includes `dig` command

```bash
yay -S bind
```

## Install Tmux Plugin

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

## Install Fisher

```bash
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher

fisher update
nvm install latest
```

## Install Gup for golang

```bash
go install github.com/nao1215/gup@latest
```

## Install Steam

```bash
# Uncomment /etc/pacman.conf
[multilib]
Include = /etc/pacman.d/mirrorlist

yay -S steam
```
