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
/srv/share 172.8.249.250/32(rw,sync,root_squash)
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
```
