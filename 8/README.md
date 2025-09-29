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
Успех
<img src='https://github.com/Makiaveli/homework/blob/main/8/Screenshot_261.jpg'>
