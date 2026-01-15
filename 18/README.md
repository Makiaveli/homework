## Vagrant

### Задание

#### 1. Подготовка окружения:

<ul>
  <li>Создать директорию для проекта.</li>
  <li>Создать базовую виртуальную машину:</li>
  <li>Настроить память ВМ: 1024 МБ.</li>
  <li>Добавить пару виртуальных диска размером 1 ГБ каждый.</li>
  <li>Настроить проброс 80 порта с гостевой системы на порт 8080 хостовой системы.</li>
  <li>Провижининг:</li>
   <ul>
     <li>Форматирует добавленные диски в файловую систему ext4.</li>
     <li>Создает точки монтирования /mnt/disk1 и /mnt/disk2.</li>
     <li>Монтирует диски в указанные директории.</lli>
     <li>Добавляет записи в /etc/fstab для автоматического монтирования при загрузке.</li>
   </ul>
</ul>

#### Проверка

```bash
[vagrant@localhost ~]$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0  40G  0 disk
`-sda1   8:1    0  40G  0 part /
sdb      8:16   0   1G  0 disk
sdc      8:32   0   1G  0 disk
```
```
[vagrant@localhost ~]$ df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  489M     0  489M   0% /dev
tmpfs          tmpfs     496M     0  496M   0% /dev/shm
tmpfs          tmpfs     496M  6.7M  489M   2% /run
tmpfs          tmpfs     496M     0  496M   0% /sys/fs/cgroup
/dev/sda1      xfs        40G  3.0G   38G   8% /
/dev/sdb       ext4      976M  2.6M  907M   1% /mnt/disk1
/dev/sdc       ext4      976M  2.6M  907M   1% /mnt/disk2
tmpfs          tmpfs     100M     0  100M   0% /run/user/1000
```

```bash
hwuser@hwlab:~$ sudo netstat -tulpn | grep 8080
[sudo] пароль для hwuser:
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      10676/VBoxHeadless
```
