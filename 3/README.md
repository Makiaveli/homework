Определяемся с устройствами
```
root@hwlab:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    2G  0 disk
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    2G  0 disk
sde                         8:64   0    2G  0 disk
sr0                        11:0    1  2,6G  0 rom
```
Размечаем sdb
```
root@hwlab:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
Создаем группу томов laba3 на sdb
```
root@hwlab:~# vgcreate laba3 /dev/sdb
  Volume group "laba3" successfully created
```
Создаем логический том testlab
```
root@hwlab:~# lvcreate -l+80%FREE -n testlab laba3
  Logical volume "testlab" created.
```
Проверяем что получилось
```
root@hwlab:~# vgdisplay laba3
  --- Volume group ---
  VG Name               laba3
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <2,00 GiB
  PE Size               4,00 MiB
  Total PE              511
  Alloc PE / Size       408 / 1,59 GiB
  Free  PE / Size       103 / 412,00 MiB
  VG UUID               XmlNFO-0dRh-DtFh-i5EU-aE7b-BQGZ-3ot21r

```
создаем файловую систему 
```
root@hwlab:~# mkfs.ext4 /dev/laba3/testlab
mke2fs 1.47.0 (5-Feb-2023)
/dev/laba3/testlab contains a btrfs file system
Proceed anyway? (y,N) y
Discarding device blocks: done
Creating filesystem with 417792 4k blocks and 104624 inodes
Filesystem UUID: f5a88e74-7d99-4fcd-a2c2-53fce9d47ac4
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

```
