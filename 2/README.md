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
veles@hwlab:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid1 sdc[1] sdb[0]
      5237760 blocks super 1.2 [2/2] [UU]

unused devices: <none>
veles@hwlab:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid1 sdc[1] sdb[0]
      5237760 blocks super 1.2 [2/2] [UU]

```
## Ломаем, чиним raid
Переводим диск sdb в состояния аварии
```
veles@hwlab:~$ mdadm /dev/md0 --fail /dev/sdb
veles@hwlab:~$ mdadm: set /dev/sdb faulty in /dev/md0

```

