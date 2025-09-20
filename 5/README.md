### 1 Определить алгоритм с наилучшим сжатием:
### 2 Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
### 3 создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.
Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
    - размер хранилища;
    - тип pool;
    - значение recordsize;
    - какое сжатие используется;
    - какая контрольная сумма используется.
Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.


Установим пакет утилит для ZFS:
```
hwuser@hwstend:~$ sudo apt install zfsutils-linux  -y
```
Создаём 4 пула по два диска в режиме RAID 1:
```
root@hwstend:~# zpool create pool1 mirror /dev/sdb /dev/sdc
root@hwstend:~# zpool create pool2 mirror /dev/sdd /dev/sde
root@hwstend:~# zpool create pool3 mirror /dev/sdf /dev/sdg
root@hwstend:~# zpool create pool4 mirror /dev/sdh /dev/sdi

```
Получилось 4 пула по 960М
```
root@hwstend:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
pool1   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool2   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool3   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool4   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -

```
Добавим разные алгоритмы сжатия в каждую файловую систему lzjb, lz4, gzip, zle:
```
root@hwstend:~# zfs set compression=lzjb pool1
root@hwstend:~# zfs set compression=lz4 pool2
root@hwstend:~# zfs set compression=gzip pool3
root@hwstend:~# zfs set compression=zle pool4

```
Проверяем
```
root@hwstend:~# zfs get all | grep compression
pool1  compression           lzjb                       local
pool2  compression           lz4                        local
pool3  compression           gzip                       local
pool4  compression           zle                        local

```
Скачиваем файл
```
root@hwstend:~# for i in {1..4}; do wget -P /pool$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```
Проверяем
```
root@hwstend:~# zfs list 
NAME    USED  AVAIL  REFER  MOUNTPOINT
pool1   148K   832M  28.5K  /pool1
pool2   146K   832M    28K  /pool2
<b>pool3   144K   832M    27K  /pool3</b>
pool4   150K   832M    31K  /pool4
```
