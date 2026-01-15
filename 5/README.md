## 1 Определить алгоритм с наилучшим сжатием
<ul>
   <li>создать 4 файловых системы на каждой применить свой алгоритм сжатия;</li>
   <li>для сжатия использовать либо текстовый файл, либо группу файлов.</li>
   <li>Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4)</li>
</ul>

Установим пакет утилит для ZFS:
```bash
hwuser@hwstend:~$ sudo apt install zfsutils-linux  -y
```
Создаём 4 пула по два диска в режиме RAID 1:
```bash
root@hwstend:~# zpool create pool1 mirror /dev/sdb /dev/sdc
root@hwstend:~# zpool create pool2 mirror /dev/sdd /dev/sde
root@hwstend:~# zpool create pool3 mirror /dev/sdf /dev/sdg
root@hwstend:~# zpool create pool4 mirror /dev/sdh /dev/sdi

```
Получилось 4 пула по 960М
```bash
root@hwstend:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
pool1   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool2   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool3   960M   106K   960M        -         -     0%     0%  1.00x    ONLINE  -
pool4   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -

```
Добавим разные алгоритмы сжатия в каждую файловую систему lzjb, lz4, gzip, zle:
```bash
root@hwstend:~# zfs set compression=lzjb pool1
root@hwstend:~# zfs set compression=lz4 pool2
root@hwstend:~# zfs set compression=gzip pool3
root@hwstend:~# zfs set compression=zle pool4

```
Проверяем
```bash
root@hwstend:~# zfs get all | grep compression
pool1  compression           lzjb                       local
pool2  compression           lz4                        local
pool3  compression           gzip                       local
pool4  compression           zle                        local

```
Скачиваем файл
```bash
root@hwstend:~# for i in {1..4}; do wget -P /pool$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```
Проверяем
```bash
root@hwstend:~# zfs list 
NAME    USED  AVAIL  REFER  MOUNTPOINT
pool1   148K   832M  28.5K  /pool1
pool2   146K   832M    28K  /pool2
pool3   144K   832M    27K  /pool3
pool4   150K   832M    31K  /pool4

root@hwstend:~# zfs get all | grep compressratio | grep -v ref
pool1  compressratio         1.04x                      -
pool2  compressratio         1.05x                      -
pool3  compressratio         1.07x                      -
pool4  compressratio         1.00x                      -

```
Таким образом, у нас получается, что алгоритм gzip-9 самый эффективный по сжатию. 
## 2 Определение настроек пула
<ul>
   <li>Определить настройки пула.</li>
   <li>собрать pool ZFS.</li>
   <li>Командами zfs определить настройки:</li>
   <ul>
      <li>- размер хранилища;</li>
      <li>- тип pool;</li>
      <li>- значение recordsize;</li>
      <li> - какое сжатие используется;</li>
      <li>- какая контрольная сумма используется.</li>
   </ul>
</ul>

<p>Скачиваем архив из методички и распаковываем его</p>

```bash
root@hwstend:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
root@hwstend:~# tar -xzvf archive.tar.gz
```
<p>Проверим, возможно ли импортировать данный каталог в пул:</p>

```bash
root@hwstend:~# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
	(Note that they may be intentionally disabled if the
	'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
	some features will not be available without an explicit 'zpool upgrade'.
 config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE

```
<p>Сделаем импорт данного пула к нам в ОС и посмотрим информацию о составе импортированного пула</p>

```bash
root@hwstend:~#  zpool import -d zpoolexport/ otus
root@hwstend:~# zpool status
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
config:

	NAME                         STATE     READ WRITE CKSUM
	otus                         ONLINE       0     0     0
	  mirror-0                   ONLINE       0     0     0
	    /root/zpoolexport/filea  ONLINE       0     0     0
	    /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

```
<p>Размер хранилища;</p>

```bash
root@hwstend:~# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -

```

<p>Тип pool;</p>

```bash
root@hwstend:~# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
```

<p>Значение recordsize;</p>

```bash
root@hwstend:~# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

```

<p>Алгоритм сжатие;</p>

```bash
root@hwstend:~# zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
```
<p>Контрольная сумма</p>

```bash
root@hwstend:~# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```
## 3 Работа со снапшотом, поиск сообщения от преподавателя
<ul>
	<li>скопировать файл из удаленной директории;</li>
	<li>восстановить файл локально. zfs receive;</li>
	<li>найти зашифрованное сообщение в файле secret_message.</li>
</ul>
	


<p>Скачаем файл, указанный в задании:</p>

```bash
root@hwstend:~# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
```

<p>Восстановим файловую систему из снапшота:</p>

```bash
root@hwstend:~# zfs receive otus/test@today < otus_task2.file
```

<p>Далее, ищем в каталоге /otus/test файл с именем “secret_message”:</p>

```bash
root@hwstend:~# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
root@hwstend:~# cat /otus/test/task1/file_mess/secret_message 
https://otus.ru/lessons/linux-hl/

```
