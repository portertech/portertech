---
title: "NFS root & OverlayFS"
date: 2023-03-04
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
to acquired six nvidia A4000 single card slot GPUs from eBay at a
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
host that provided them PXE boot services - perfect! Now, I knew it
was fairly common to mount a NFS root file system during the boot
process, however, I didn't want the NFS export to be mutable. I wanted
a single (shared) NFS export to act like a simple boot image that can
be centrally managed. Thanks to Docker, I have become more familiar
with OverlayFS, a file system that allows a directory tree to be
overlaid onto another. OverlayFS is typically used for live CDs or
container file systems, a writable layer over a read-only one. Another
file system that's typically applied in these cases is Tmpfs, a system
which keeps files in virtual memory (temporary/volatile). I wanted an
OverlayFS root that used a writable Tmpfs layer over a remote
read-only NFS export!
