---
title: "NFS root & OverlayFS"
date: 2023-03-06
draft: false
toc: false
images:
tags:
  - linux
  - rocky
  - metal
  - datacenter
  - nfs
  - overlayfs
  - network
  - boot
  - sysadmin
  - devops
---

Thanks to the fairly recent "crypto collapse" and ever increasing
crypto network difficulty, used GPUs are back on the menu! I managed
to acquire six nvidia A4000 single card slot GPUs from eBay at a
bargain price. Crypto miners were dumping these sweet cards just in
time for me to experiment with GPU acceleration of all-the-things. My
home lab already had three identical nodes ready to house the GPUs,
but they were in need of some tender loving care. The nodes needed a
fresh OS install but I didn't want to PXE boot Anaconda and use local
drives this time around. Having wanting to try a diskless deployment
for some time, this was the perfect opportunity to tinker!

### Read-Only NFS Root!

The Network File System is a distributed file system protocol that
allows a user on a remote host to access files over the network. The
three nodes already had access to a private network which included a
host which provided them PXE boot services - perfect! Now, I knew it
was fairly common to mount a NFS root file system during the boot
process, however, I didn't want the NFS export to be mutable. I wanted
a single (shared) NFS export to act like a simple boot image that can
be centrally managed. Thanks to Docker, I have become more familiar
with OverlayFS, a file system that allows a directory tree to be
overlaid onto another. OverlayFS is typically used for live CDs or
container file systems, a writable layer over a read-only one. Another
file system that's typically applied in these cases is tmpfs, a system
which keeps files in virtual memory (temporary/volatile). I wanted an
OverlayFS root that used a writable tmpfs layer over a remote
read-only NFS export! Luckily for me, someone else had already figured
it out :)

### Instructions

After a little searching, I discovered a post by "voleg" with
instructions to setup diskless RHEL 8.5, including... "the idea of
mounting an overlay filesystem as root"! Considering that I wanted to
use Rocky Linux 9, this was very promising, and the instructions
_almost worked_ for me.

Here is the post that got me 90% of the way: [Diskless REDHAT (on Red
Hat 8.5)](http://www.voleg.info/redhat-diskless.html). I won't
duplicate their content here as I only had to make a few minor
tweaks. **SO CLOSE!**

#### Tweaks

_NOTE: You will likely need to alter these steps for your own
environment (e.g. network addresses, etc)._

When generating the initrd (`rocky9-nfs.ird` in my case), I had to
include a few additional drivers and modules. I am sure the list can
be further reduced, however, I didn't want to take the time to
optimize here.

```
dracut --no-hostonly --no-hostonly-cmdline --nofscks \
--add-drivers "virtio_net overlay nfs nfsv4" \
--install more -m "bash systemd systemd-initrd i18n network \
ifcfg dm kernel-modules kernel-modules-extra kernel-network-modules \
rootfs-block terminfo udev-rules dracut-systemd usrmount base fs-lib nfs" \
--force /boot/rocky9-nfs.ird 5.14.0-162.12.1.el9_1.0.2.x86_64
```

In order to boot UEFI nodes, I had to serve up a few additional files
via TFTP and change the DHCP configuration.

```
yum install --downloadonly --downloaddir=/root/packages shim-x64 grub2-efi-x64.x86_64

cd /root/packages

rpm2cpio shim-x64-* | cpio -dmi --quiet
rpm2cpio grub2-efi-x64-* | cpio -dmi --quiet

cp boot/efi/EFI/rocky/shim.efi /tftpboot/
cp boot/efi/EFI/rocky/grubx64.efi /tftpboot/
```

```
nano /tftpboot/grub.cfg
```

`/tftpboot/grub.cfg`

```
set default=0
set timeout=3

menuentry 'NFS root' --class fedora --class gnu-linux --class gnu --class os {
   linuxefi rocky9-nfs.krl splash=none root=nfs:192.168.2.2:/export/root:ro,vers=4.2,sec=sys,nolock net.ifnames=0 selinux=0
   initrdefi rocky9-nfs.ird
}
```

```
nano /etc/dhcp/dhcpd.conf
```

`/etc/dhcp/dhcpd.conf`

```
allow booting;
allow bootp;
ddns-update-style none;
default-lease-time 31556952; # one year
max-lease-time 31556952; # otherwise the static assignment expires at 24h and the NFS root breaks.
ignore client-updates;
update-static-leases on;

option space pxelinux;
option pxelinux.magic code 208 = string;
option pxelinux.configfile code 209 = text;
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option architecture-type code 93 = unsigned integer 16;

subnet 192.168.2.0 netmask 255.255.255.0 {
        interface ens19;

        option subnet-mask 255.255.255.0;

        next-server 192.168.2.2;

        if option architecture-type = 00:07 {
                filename "shim.efi";
        } else {
                filename "pxelinux.0";
        }

        pool {
                range dynamic-bootp 192.168.2.20 192.168.2.254;

                host node1 {
                        hardware ethernet 68:05:CA:93:95:A5;
                        fixed-address 192.168.2.5;
                }

                host node2 {
                        hardware ethernet 68:05:CA:93:95:BE;
                        fixed-address 192.168.2.6;
                }

                host node3 {
                        hardware ethernet 68:05:CA:93:95:A1;
                        fixed-address 192.168.2.7;
                }
        }
}
```

```
systemctl restart dhcpd.service
```

### The Experience

So far I have been very pleased with the deployment. The NFS root has
made managing the hosts a breeze, including updating nvidia drivers
etc. Having node local changes being completely ephemeral helps avoid
one experiment from impacting another - a clean slate is just a reboot
away! I'll try to follow-up with a post about my centralized logging
and monitoring for these nodes.
