# Файловые системы и LVM-1

Определяемся с устройствами
```bash
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
```bash
root@hwlab:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
Создаем группу томов laba3 на sdb
```bash
root@hwlab:~# vgcreate laba3 /dev/sdb
  Volume group "laba3" successfully created
```
Создаем логический том testlab
```bash
root@hwlab:~# lvcreate -l+80%FREE -n testlab laba3
  Logical volume "testlab" created.
```
Проверяем что получилось
```bash
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
```bash
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
Проверяем
```bash
lsblk -f | grep 'laba3-testlab'
└─laba3-testlab           ext4        1.0                                              f5a88e74-7d99-4fcd-a2c2-53fce9d47ac4
```
Создадим папку data и примонтируем ее к созданому lvm
```bash
root@hwlab:~# mkdir /data
root@hwlab:~# mount /dev/laba3/testlab /data/
```

## для того, что расширить lvm testlab размечаечаем sdc
```
root@hwlab:~# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```
Добавляем sdc в группу laba3
```
root@hwlab:~# vgextend laba3 /dev/sdc
  Volume group "laba3" successfully extended
```
Проверяем
```
root@hwlab:~# vgdisplay -v laba3 | grep 'PV Name'
  PV Name               /dev/sdb
  PV Name               /dev/sdc
root@hwlab:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  laba3       2   1   0 wz--n-   3,99g <2,40g
  ubuntu-vg   1   1   0 wz--n- <14,25g <4,25g

```
Сымитируем занятое место с помощью команды dd
```
root@hwlab:~# dd if=/dev/zero of=/data/test.log bs=1M \
 count=8000 status=progress

1613758464 bytes (1,6 GB, 1,5 GiB) copied, 27 s, 59,8 MB/s
dd: error writing '/data/test.log': No space left on device
1554+0 records in
1553+0 records out
1628438528 bytes (1,6 GB, 1,5 GiB) copied, 27,3444 s, 59,6 MB/s
```
Проверяем
```
root@hwlab:~# df -Th /data/
Filesystem                Type  Size  Used Avail Use% Mounted on
/dev/mapper/laba3-testlab ext4  1,6G  1,6G     0 100% /data
```
Увеличиваем LV за счет появившегося свободного места
```
root@hwlab:~# lvextend -l+100%FREE /dev/laba3/testlab
  Size of logical volume laba3/testlab changed from 1,59 GiB (408 extents) to 3,99 GiB (1022 extents).
  Logical volume laba3/testlab successfully resized.

```
Произведем resize файловой системы:
```
root@hwlab:~# resize2fs /dev/laba3/testlab
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/laba3/testlab is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/laba3/testlab is now 1046528 (4k) blocks long.
```
Проверяем
```
root@hwlab:~# df -Th /data
Filesystem                Type  Size  Used Avail Use% Mounted on
/dev/mapper/laba3-testlab ext4  3,9G  1,6G  2,2G  41% /data
```
