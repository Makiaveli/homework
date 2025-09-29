## Работа с NFS 
<p>Устанавливаем пакет с NFS сервером</p>

```
root@hwsrv:~# apt install nfs-kernel-server
```
<p>Создадим директорию которую в последствии будем использовать, изменим владельца и поменяем права на папук</p>

```
root@hwsrv:~# mkdir -p /srv/share/upload
root@hwsrv:~# chown -R nobody:nogroup /srv/share
root@hwsrv:~# chmod 0777 /srv/share/upload
```
<p>Редактируем файл /etc/exports</p>
<p>Предоставим доступ к директории хосту 172.8.249.250</p>
<p>Выдадим права на чтение/запись</p>
<p>Настроим согласованность</p>
<p>Запретим использовать права root на сервере</p>

```
root@hwsrv:~# cat << EOF > /etc/exports 
/srv/share 172.18.249.250/32(rw,sync,root_squash)
EOF

```
<p>Экспортируем созданную директорию и проверяем</p>

```
root@hwsrv:~# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "172.8.249.250/32:/srv/share".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x

root@hwsrv:~# exportfs -s
/srv/share  172.8.249.250/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)

```
<p>На клиенте устанавливаем паке nfs-common</p>

```
hwuser@hwstend:~$ sudo apt install nfs-common -y
```
<p>В /etc/fstab добавляем точку монтирования используя systemd для автоматического монтирования при обращении к точке монтирования </p>

```
root@hwstend:~# echo "172.18.249.249:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
```
<p>активируем и применяем изменения в конфигурации монтирования NFS-шары</p>

```
root@hwstend:~# systemctl daemon-reload
root@hwstend:~# systemctl restart remote-fs.target
```
<p>Проверяем</p>

```
hwuser@hwstend:/mnt/upload$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=64,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=2753)
172.18.249.249:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=172.18.249.249,mountvers=3,mountport=52620,mountproto=udp,local_lock=none,addr=172.18.249.249)

```
## Проверяем рабоотоспособность
### Предварительно проверяем клиент:
<p>Создаем файл на nfs шаре</p>

```
root@hwsrv:~# touch /srv/share/upload/check_file
```
<p>Перезагружаем проверяем с клиента доступ к файлу</p>

```
hwuser@hwstend:/mnt/upload$ ll /mnt/upload/
total 8
drwxrwxrwx 2 nobody nogroup 4096 сен 29 14:06 ./
drwxr-xr-x 3 nobody nogroup 4096 сен 29 12:45 ../
-rw-r--r-- 1 root   root       0 сен 29 14:06 check_file

```
### Проверяем сервер:
<p>перезагружаем сервер, проверяем наличие файлов в каталоге /srv/share/upload/</p>

```
hwuser@hwsrv:~$ ll /srv/share/upload/
total 8
drwxrwxrwx 2 nobody nogroup 4096 сен 29 14:06 ./
drwxr-xr-x 3 nobody nogroup 4096 сен 29 12:45 ../
-rw-r--r-- 1 root   root       0 сен 29 14:06 check_file
```
<p>проверяем экспорты exportfs -s;</p>

```
hwuser@hwsrv:~$ sudo exportfs -s
[sudo] password for hwuser:
/srv/share  172.18.249.250/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)

```
<p>проверяем работу RPC showmount -a 172.18.249.250</p>

```
hwuser@hwsrv:~$ sudo showmount -a 172.18.249.249
All mount points on 172.18.249.249:
172.18.249.250:/srv/share
```

### Проверяем клиент

<p>Перзагружаемся, прверяем showmount</p>

```
hwuser@hwstend:~$ sudo showmount -a 172.18.249.249
[sudo] password for hwuser:
All mount points on 172.18.249.249:
172.18.249.250:/srv/share
```
<p>Переходим в каталог /mnt/upload и проверяем статус монтирования </p>

```
hwuser@hwstend:~$ cd /mnt/upload/
hwuser@hwstend:/mnt/upload$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=64,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=4772)
172.18.249.249:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=172.18.249.249,mountvers=3,mountport=54760,mountproto=udp,local_lock=none,addr=172.18.249.249)
```

<p>Создаем текстовый файл и проверяем, что он успешно создан</p>

```
hwuser@hwstend:/mnt/upload$ touch final_check
hwuser@hwstend:/mnt/upload$ ll
total 8
drwxrwxrwx 2 nobody nogroup 4096 сен 29 14:33 ./
drwxr-xr-x 3 nobody nogroup 4096 сен 29 12:45 ../
-rw-r--r-- 1 root   root       0 сен 29 14:06 check_file
-rw-rw-r-- 1 hwuser hwuser     0 сен 29 14:33 final_check

```
```
hwuser@hwsrv:~$ ll /srv/share/upload/
total 8
drwxrwxrwx 2 nobody nogroup 4096 сен 29 14:33 ./
drwxr-xr-x 3 nobody nogroup 4096 сен 29 12:45 ../
-rw-r--r-- 1 root   root       0 сен 29 14:06 check_file
-rw-rw-r-- 1 veles  veles      0 сен 29 14:33 final_check

```
