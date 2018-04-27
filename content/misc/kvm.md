---
title: "KVM"
date: 2018-01-16 17:28:48
---
[TOC]

## List all the KVM domain(machine name)
```ksh
# virsh list --all
 Id    Name                           State
 ----------------------------------------------------
  -     win7_x64-KVM                   shut off
  -
```

## Start/shutdown the KVM

```ksh
# virsh start win7_x64-KVM
Domain win7_x64-KVM started
# virsh shutdown win7_x64-KVM
Domain win7_x64-KVM is being shutdown

```


## Getting the volume information
```ksh
# virsh vol-list --pool default
Name                 Path
-----------------------------------------
win7_x64-KVM.img     /var/lib/libvirt/images/win7_x64-KVM.img
win7_x64-KVM.qcow2   /var/lib/libvirt/images/win7_x64-KVM.qcow2
win7_x64-KVM.qcow2.type /var/lib/libvirt/images/win7_x64-KVM.qcow2.type
```


## Removing the volume

```ksh
# virsh vol-delete --pool default win7_x64-KVM.img
Vol win7_x64-KVM.img deleted

```

## Extending the file system

```ksh
cd /var/lib/libvirt/clone
# cp Virtual_Client_for_Linux_KVM_Windows_7.qcow2 Virtual_Client_for_Linux_KVM_Windows_7.qcow2.bak
virsh vol-resize win7_x64-KVM.qcow2 35G --pool default
Size of volume 'win7_x64-KVM.qcow2' successfully changed to 35G
```


# Deleting the whole KVM guest
```ksh
virsh shutdown hostname
virsh list --all
virsh undifine hostname
virsh vol-delete --pool default win7_x64-KVM.img
```

# Extending disk space on VM guest
```bash
# virsh vol-list --pool default
Name                 Path                                    
-----------------------------------------
win7_x64-KVM.qcow2   /var/lib/libvirt/images/win7_x64-KVM.qcow2
win7_x64-KVM.qcow2.type /var/lib/libvirt/images/win7_x64-KVM.qcow2.type

# qemu-img info /var/lib/libvirt/images/win7_x64-KVM.qcow2
image: /var/lib/libvirt/images/win7_x64-KVM.qcow2
file format: qcow2
virtual size: 35G (37580963840 bytes)
disk size: 32G
cluster_size: 65536

# qemu-img resize /var/lib/libvirt/images/win7_x64-KVM.qcow2 +5G
Image resized.
root@oc7421025535 ~
# qemu-img info /var/lib/libvirt/images/win7_x64-KVM.qcow2
image: /var/lib/libvirt/images/win7_x64-KVM.qcow2
file format: qcow2
virtual size: 40G (42949672960 bytes)
disk size: 32G
cluster_size: 65536

```


