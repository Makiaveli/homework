## Создаем raid
Cмотрим, какие блочные устройства у нас есть
```
userhw@hwlab:~$ lsnlk

userhw@hwlab:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    5G  0 disk
sdc                         8:32   0    5G  0 disk
sr0                        11:0    1  2,6G  0 rom

```
Зануляем суперблоки
```
userhw@hwlab:~$ sudo mdadm --zero-superblock --force /dev/sd{b,c}

```
Создаем 1 рейд из двух дисков sdb, sdc
```
userhw@hwlab:~$ mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b,c}
```
Проверяем
```
userhw@hwlab:~$ sudo mdadm -D /dev/md0
[sudo] password for veles:
/dev/md0:
           Version : 1.2
     Creation Time : Sat Sep  6 22:08:37 2025
        Raid Level : raid1
        Array Size : 5237760 (5.00 GiB 5.36 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Sep  7 04:40:50 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : hwlab:0  (local to host hwlab)
              UUID : 94dd2eee:e61d8973:b619d76a:d6fa9362
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc


```
## Ломаем, чиним raid
Переводим диск sdb в состояния аварии
```
veles@hwlab:~$ sudo mdadm /dev/md0 --fail /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0

```
Проверяем
```
veles@hwlab:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid1 sdc[1] sdb[0](F)
      5237760 blocks super 1.2 [2/1] [_U]

```
```
veles@hwlab:~$ sudo  mdadm -D /dev/md0

/dev/md0:
           Version : 1.2
     Creation Time : Sat Sep  6 22:08:37 2025
        Raid Level : raid1
        Array Size : 5237760 (5.00 GiB 5.36 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Sep  7 06:09:32 2025
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : hwlab:0  (local to host hwlab)
              UUID : 94dd2eee:e61d8973:b619d76a:d6fa9362
            Events : 22

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync   /dev/sdc

       0       8       16        -      faulty   /dev/sdb
```
Удаляем сломанный диск sdb из массива
```
veles@hwlab:~$ sudo mdadm /dev/md0 --remove /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
```
Добавляем в рейд "новый" диск  sdd взамен "вышедшего из строя диска" sdb
```
sudo mdadm /dev/md0 --add /dev/sdd
mdadm: cannot load array metadata from /dev/md0
```
Проверяем
```
veles@hwlab:~$ sudo  mdadm -D /dev/md127
/dev/md0:
           Version : 1.2
     Creation Time : Sat Sep  6 22:08:37 2025
        Raid Level : raid1
        Array Size : 5237760 (5.00 GiB 5.36 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Sep  7 06:34:09 2025
             State : clean, degraded, recovering
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

    Rebuild Status : 4% complete

              Name : hwlab:0  (local to host hwlab)
              UUID : 94dd2eee:e61d8973:b619d76a:d6fa9362
            Events : 26

    Number   Major   Minor   RaidDevice State
       2       8       48        0      spare rebuilding   /dev/sdd
       1       8       32        1      active sync   /dev/sdc
veles@hwlab:~$ cat /proc/mdstat
```
```
Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid1 sdd[2] sdc[1]
      5237760 blocks super 1.2 [2/1] [_U]
      [====>................]  recovery = 22.9% (1205120/5237760) finish=1.7min speed=37660K/sec

unused devices: <none>

```
