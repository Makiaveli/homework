
Запускаем ВМ и смотрим текущее ядро
```
hwuser@hwlab:~$ uname -r
6.8.0-79-generic


```
Создаем директорию, переходим в нее и скачиваем пакеты
```
hwuser@hwlab:~$ mkdir kernel && cd kernel
hwuser@hwlab:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.17-rc4/amd64/linux-headers-6.17.0-061700rc4-generic_6.17.0-061700rc4.202508312336_amd64.deb
hwuser@hwlab:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.17-rc4/amd64/linux-headers-6.17.0-061700rc4_6.17.0-061700rc4.202508312336_all.deb
hwuser@hwlab:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.17-rc4/amd64/linux-image-unsigned-6.17.0-061700rc4-generic_6.17.0-061700rc4.202508312336_amd64.deb
hwuser@hwlab:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.17-rc4/amd64/linux-modules-6.17.0-061700rc4-generic_6.17.0-061700rc4.202508312336_amd64.deb
```

Устанавливаем пакеты
```
hwuser@hwlab:~/kernel$ sudo dpkg -i *.deb 
```

Обновляем конф-ию загрузчика, выбираем загрузку нового ядра по умолчанию
```
hwuser@hwlab:~/kernel$ sudo update-grub
hwuser@hwlab:~/kernel$  sudo grub-set-default 0
```
Перезагружаемся, проверяем.
```
hwuser@hwlab:~$ uname -r
6.17.0-061700rc4-generic
```
