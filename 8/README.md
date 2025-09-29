## Включить отображение меню Grub
<p>Редактируем файл /etc/default/grub</p>
<p>Комментируем строку, скрывающую меню и ставим задержку для выбора пункта меню в 10 секунд.</p>
<p></p>#GRUB_TIMEOUT_STYLE=hidden</p>
<p>GRUB_TIMEOUT=10
</p>
```
hwuser@hwstend:/$ sudo nano /etc/default/grub
```
<p>Обновляем конфигурацию загрузчика и перезагружаемся для проверки.</p>

```
hwuser@hwstend:/$ sudo update-grub
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.0-84-generic
Found initrd image: /boot/initrd.img-6.8.0-84-generic
Found linux image: /boot/vmlinuz-6.8.0-83-generic
Found initrd image: /boot/initrd.img-6.8.0-83-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
hwuser@hwstend:/$ sudo reboot

```
<p>Успех</p>
<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_261.jpg'>

## Попасть в систему без пароля несколькими способами

### Способ 1. init=/bin/bash

<p>В конце строки, начинающейся с linux, добавляем init=/bin/bash и нажимаем сtrl-x для загрузки в систему</p>

<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_262.jpg'>
<p>Проверяем</p>
<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_263.jpg'>

### Способ 2. Recovery mode
<p>В меню загрузчика на первом уровне выбрать второй пункт (Advanced options…), далее загрузить пункт меню с указанием recovery mode в названии. 
Получим меню режима восстановления.
</p>
<p>В этом меню сначала включаем поддержку сети (network) для того, чтобы файловая система перемонтировалась в режим read/write (либо это можно сделать вручную).
Далее выбираем пункт root и попадаем в консоль с пользователем root. Если вы ранее устанавливали пароль для пользователя root (по умолчанию его нет), то необходимо его ввести. 
В этой консоли можно производить любые манипуляции с системой.
</p>
<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_264.jpg'>

## Установить систему с LVM, после чего переименовать VG

<p>Первым делом посмотрим текущее состояние системы (список Volume Group):
</p>

```
hwuser@hwstend:~$ sudo vgs
[sudo] password for hwuser:
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <22,00g 11,00g

```
<p>Нас интересует вторая строка с именем Volume Group. Приступим к переименованию:</p>

```
hwuser@hwstend:~$ sudo vgrename ubuntu-vg ubuntu-typeyshaya-laba
  Volume group "ubuntu-vg" successfully renamed to "ubuntu-typeyshaya-laba"
```
<p>Далее правим /boot/grub/grub.cfg. Везде заменяем старое название VG на новое (в файле дефис меняется на два дефиса ubuntu--vg ubuntu--otus).
После чего можем перезагружаться и, если все сделано правильно, успешно грузимся с новым именем Volume Group и проверяем:
</p>
<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_265.jpg'>

```
hwuser@hwstend:~$ sudo vgs
  VG                     #PV #LV #SN Attr   VSize   VFree
  ubuntu-typeyshaya-laba   1   1   0 wz--n- <22,00g 11,00g

```
