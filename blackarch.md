# BlackArch

## Repository

- [keyring](https://github.com/BlackArch/blackarch-keyring)
- [Mirror](https://github.com/BlackArch/blackarch/blob/master/mirror/mirror.lst)

## Manual install

- I got offline of website

```
git clone https://github.com/BlackArch/blackarch-keyring.git
cd blackarch-keyring
sudo pacman-key --add blackarch.gpg

# https://github.com/BlackArch/blackarch-keyring/blob/master/packager-keyids
sudo pacman-key --lsign-key 4345771566D76038C7FEB43863EC0ADBEA87E4E3
sudo pacman-key --lsign-key F6DA3545F655964DD9177918C65009B64EB7BB3C
sudo pacman-key --lsign-key F9A6E68A711354D84A9B91637533BAFE69A25079

# Edit /etc/pacman.conf
[blackarch]
Server = $mirror

sudo pacman -Syu
sudo pacman -S blackman

# optional
sudo pacman -S blackarch
```

## Tools

```sh
# Web
pacman -S seclists
pacman -S hydra gobuster

# Network
pacman -S wireshark-qt
pacman -S tcpdump

# RDP Remote
pacman -S remmina

# Reverse Engineer
pacman -S ghidra
```
