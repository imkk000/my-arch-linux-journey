# QEMU VM Configuration

## Prerequisite

- [*] qemu-full - hypervisor
- [virt-manager](https://wiki.archlinux.org/title/Virt-manager) - gui
- libvirt - backend for hypervisor management
- dnsmasq - dns server

## Manual Step

- Create virtual network ([bridge](https://wiki.archlinux.org/title/Network_bridge))
- [UEFI](https://www.qemu.org/docs/master/devel/uefi-vars.html)
- [Tails Guideline](https://tails.net/doc/advanced_topics/virtualization/virt-manager/index.en.html)
- Set memory size: `-m mem_size (4G, 4096)`
- Set CPU `-smp 4 (core)` or `-smp 16,sockets=2,clusters=2,cores=2,threads=2,maxcpus=16`
- Monitor mode swap (ctrl+alt+1 and ctrl+alt+2)

```
# all save inside qcow2
savevm snapshot1     # create
loadvm snapshot1     # restore
info snapshots       # list
delvm snapshot1      # delete
quit                 # stop VM
```

## Useful Command

```bash
# create virtual network (bridge)
ip link add name br0 type bridge
ip link set dev br0 up

# edit allow network
cat > /etc/qemu/bridge.conf << EOF
allow br0
EOF

# boot order
-boot c (cd-rom)
-boot d (disk)
-boot n (network)
-boot a (floppy)

# use default bridge
qemu-system-x86_64 -m 4096 -smp 4 -nic bridge -cdrom <iso> -display gtk

# use default nat
qemu-system-x86_64 -no-defaults -no-user-config -display gtk -cpu host -m 4096 -smp 4 \
    -netdev user,id=net0,restrict=on -device virtio-net,netdev=net0 \
    -drive file=vm.qcow2,format=qcow2,if=virtio \
    -cdrom <iso>

# hardening vm
qemu-system-x86_64 \
  # --- CPU & RAM ---
  -cpu host \
  -smp 2 \
  -m 4G \
  -enable-kvm \
  # --- BIOS ---
  -bios /usr/share/ovmf/x64/OVMF.4m.fd \
  \
  # --- Disk ---
  -drive file=vm.qcow2,format=qcow2,if=virtio \
  \
  # --- CDROM (remove after install) ---
  -drive file=blackarch.iso,media=cdrom,readonly=on \
  -boot order=dc,once=d \
  \
  # --- Network (NAT, isolated) ---
  -netdev user,id=net0,restrict=on \
  -device virtio-net,netdev=net0 \
  \
  # --- Display ---
  -display gtk \
  -vga virtio \
  \
  # --- Hardening: strip defaults ---
  -nodefaults \
  -no-user-config \
  \
  # --- QMP socket ---
  -qmp unix:/tmp/qmp-vm.sock,server,nowait \
  \
  # --- Seccomp ---
  -sandbox on,obsolete=deny,elevateprivileges=deny,spawn=deny,resourcecontrol=deny \
  \
  # --- Monitor ---
  -monitor vc
  \
  # --- Mount Host to VM ---
  -virtfs local,path=/host/dir,mount_tag=hostshare,security_model=passthrough
  # inside: mount -t 9p -o trans=virtio hostshare /mnt/host
```

## Snapshot Disk (qcow2)

- must stop vm before snapshot

```bash
# increase raw image size
truncate -s size tails.img

qemu-img create -f qcow2 disk.qcow2 8G

# clone disk (-b linked)
qemu-img create -f qcow2 -F qcow2 -b disk.qcow2 clone.qcow2

# clone from specific snapshot (-l)
qemu-img convert -f qcow2 -O qcow2 -l updated base.qcow2 clone.qcow2

# create
qemu-img snapshot -c name disk.qcow2

# list
qemu-img snapshot -l disk.qcow2

# restore
qemu-img snapshot -a name disk.qcow2

# delete
qemu-img snapshot -d name disk.qcow2

# convert vmware disk into qcow2
qemu-img convert -f vmdk -O qcow2 vmware.vmdk disk.qcow2

# resize disk
qemu-img resize disk.qcow2 +10G
qemu-img resize disk.qcow2 -10G

# fixed size
qemu-img resize disk.qcow2 50G
```
