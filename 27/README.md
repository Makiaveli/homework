## Домашнее задание Резервное копирование
<p>Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:</p>
<ul>
  <li><b>1.</b> директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)</li>
  <li><b>2.</b>репозиторий для резервных копий должен быть зашифрован ключом или паролем - на усмотрение студента;</li>
  <li><b>3.</b>имя бэкапа должно содержать информацию о времени снятия бекапа;</li>
  <li><b>4.</b>глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;</li>
  <li><b>5.</b>резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;</li>
  <li><b>6.</b>написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на усмотрение студента;</li>
  <li><b>7.</b>настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.</li>
</ul>
<hr>
### 1
#### Устанавливаем borg на клиент и сервер;
#### cоздаём отдельную точку монтирования /var/backup;
#### cоздаём пользователя под бэкапы;
#### добавляем точку монтирования в /etc/fstab

```
hwuser@hwsrv:~$ sudo apt install borgbackup -y
hwuser@hwstend:~$ sudo apt install borgbackup -y
hwuser@hwsrv:~$ fallocate -l 2g /var/backup.img
hwuser@hwsrv:~$ sudo mkfs.ext4 /var/backup.img
hwuser@hwsrv:~$ sudo mkdir -p /var/backup
hwuser@hwsrv:~$ sudo mount /var/backup.img /var/backup
hwuser@hwsrv:~$ sudo useradd -m -s /bin/bash borg
hwuser@hwsrv:~$ sudo chown borg:borg /var/backup
```

### Настройка SSH-доступа client → backup

```
hwuser@hwsrv:~$ sudo ssh-keygen -t ed25519 -f /root/.ssh/borg_backup -N ""
hwuser@hwstend:~$ sudo ssh-copy-id -i /root/.ssh/borg_backup.pub borg@172.18.249.249
hwuser@hwstend:~$ sudo borg init   --encryption=repokey-blake2   borg@172.18.249.249:/var/backup/etc_repo
```

### Скрипт резервного копирования

```
#!/bin/bash

export BORG_REPO="borg@172.18.249.249:/var/backup/etc_repo"
export BORG_RSH="ssh -i /root/.ssh/borg_backup"
export BORG_PASSPHRASE="P@$$w0RD"

TAG="borg-etc-backup"
HOSTNAME=$(hostname)
DATE=$(date +"%Y-%m-%d_%H-%M")

logger -t "$TAG" "Backup started"

borg create \
  --stats \
  --compression lz4 \
  ::${HOSTNAME}-etc-${DATE} \
  /etc \
  2>&1 | logger -t "$TAG"

if [ "${PIPESTATUS[0]}" -ne 0 ]; then
  logger -t "$TAG" "Backup failed"
  exit 1
fi

# Политика хранения:
# последние 3 месяца — ежедневно
# остальное до года — по 1 в месяц

borg prune \
  --list \
  --keep-daily=90 \
  --keep-monthly=12 \
  2>&1 | logger -t "$TAG"

borg compact 2>&1 | logger -t "$TAG"

logger -t "$TAG" "Backup finished successfully"
```
### Делаем исполняемым
```
chmod +x /usr/local/bin/borg_etc_backup.sh
```
### Systemd service и timer (каждые 5 минут)
#### Service
```
hwuser@hwstend:~$ sudo nano /etc/systemd/system/borg-etc-backup.service
```
```
[Unit]
Description=BorgBackup /etc backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/borg_etc_backup.sh
```
#### Timer
```
hwuser@hwstend:~$ sudo nano /etc/systemd/system/borg-etc-backup.timer
```
```
[Unit]
Description=Run borg /etc backup every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```

#### Активируем

```
hwuser@hwstend:~$ sudo systemctl enable borg-etc-backup.timer --now
Created symlink /etc/systemd/system/timers.target.wants/borg-etc-backup.timer → /etc/systemd/system/borg-etc-backup.timer.
```

### Проверяем 

```
hwuser@hwstend:~$ sudo systemctl list-timers | grep borg
Thu 2026-01-15 14:33:40 UTC       4min 35s Thu 2026-01-15 14:28:40 UTC      24s ago borg-etc-backup.timer          borg-etc-backup.service
```

```
hwuser@hwstend:~$ sudo journalctl -t borg-etc-backup
янв 15 14:46:13 hwstend borg-etc-backup[124766]: Backup started
янв 15 14:46:19 hwstend borg-etc-backup[124768]: ------------------------------------------------------------------------------
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Repository: ssh://borg@172.18.249.249/var/backup/etc_repo
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Archive name: hwstend-etc-2026-01-15_14-46
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Archive fingerprint: 4ee1d88e2a1b5f414cec330ac84719b6307cc4fd998a310756d3e08e61dbd235
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Time (start): Thu, 2026-01-15 14:46:16
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Time (end):   Thu, 2026-01-15 14:46:18
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Duration: 2.21 seconds
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Number of files: 860
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Utilization of max. archive size: 0%
янв 15 14:46:19 hwstend borg-etc-backup[124768]: ------------------------------------------------------------------------------
янв 15 14:46:19 hwstend borg-etc-backup[124768]:                        Original size      Compressed size    Deduplicated size
янв 15 14:46:19 hwstend borg-etc-backup[124768]: This archive:                2.34 MB              1.04 MB              1.01 MB
янв 15 14:46:19 hwstend borg-etc-backup[124768]: All archives:                2.34 MB              1.04 MB              1.09 MB
янв 15 14:46:19 hwstend borg-etc-backup[124768]:
янв 15 14:46:19 hwstend borg-etc-backup[124768]:                        Unique chunks         Total chunks
янв 15 14:46:19 hwstend borg-etc-backup[124768]: Chunk index:                     818                  851
янв 15 14:46:19 hwstend borg-etc-backup[124768]: ------------------------------------------------------------------------------
янв 15 14:46:22 hwstend borg-etc-backup[124812]: Keeping archive (rule: daily #1):        hwstend-etc-2026-01-15_14-46         Thu, 2026-01-15 14:46:16 [4ee1d88e2a1b5f414cec330ac84719b6307cc4fd998a310756d3>
янв 15 14:46:25 hwstend borg-etc-backup[124859]: Backup finished successfully

```

```
hwuser@hwstend:~$ sudo borg list borg@172.18.249.249:/var/backup/etc_repo
borg@172.18.249.249's password:
Enter passphrase for key ssh://borg@172.18.249.249/var/backup/etc_repo:
hwstend-etc-2026-01-15_14-46         Thu, 2026-01-15 14:46:16 [4ee1d88e2a1b5f414cec330ac84719b6307cc4fd998a310756d3e08e61dbd235]
```
