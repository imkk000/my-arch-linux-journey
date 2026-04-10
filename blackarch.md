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
# Editor
pacman -S code

# Fake Service
yay -S responder

# cannot noconfirm
pacman -S wireshark-qt

# Network
pacman -S wireshark-cli # tshark
pacman -S tcpdump
pacman -S nmap
pacman -S inetutils # telnet
pacman -S libparistraceroute
pacman -S whois bind
pacman -S openbsd-netcat

# Remote
pacman -S openssh
pacman -S remmina
pacman -S dbeaver
pacman -S smbclient
yay -S postman-bin

# Reverse Engineer
pacman -S ghidra

# Programming
pacman -S go
pacman -S python python-pip

# Brute Force
pacman -S hashcat
pacman -S john
pacman -S seclists
pacman -S hydra gobuster
blackman -s netexec

# https://gitlab.com/kalilinux/packages/hash-identifier/
# install at /opt/hash-identifier
kali(hash-identifier): hash-id.py

# Miscellaneous
pacman -S git tree sudo
pacman -S zip unzip unrar
pacman -S man neovim ripgrep
```
