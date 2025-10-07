## Systemd — создание unit-файла
<p> создаём файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные.</p>

```
hwuser@hwstend:~$ sudo nano /etc/default/watchlog

# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log

```

<p> Создадим скрипт:</p>

```
hwuser@hwstend:~$ sudo nano /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

```

<p>Добавим права на запуск файла:</p>

```
hwuser@hwstend:~$ sudo chmod +x /opt/watchlog.sh
```

<p>Создадим юнит для сервиса:</p>

```
hwuser@hwstend:~$ sudo nano /etc/systemd/system/watchlog.service

[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

```
<p>Создадим юнит для таймера:</p>

```
hwuser@hwstend:~$ sudo nano /etc/systemd/system/watchlog.timer

[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target


```

<p>Запускаем timer</p>

```
hwuser@hwstend:~$ sudo systemctl start watchlog.timer
```
<p>И убедиться в результате:</p>

```
hwuser@hwstend:~$ sudo tail -n 1000 /var/log/syslog  | grep word
2025-10-02T13:28:42.978021+00:00 hwstend hwuser: Чт 02 окт 2025 13:28:42 UTC: I found word, Master!
2025-10-02T13:30:03.904081+00:00 hwstend hwuser: Чт 02 окт 2025 13:30:03 UTC: I found word, typoe perepisivanie metodichki!

```

### Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта

<p>Устанавливаем spawn-fcgi и необходимые для него пакеты:</p>

```
hwuser@hwstend:~$ sudo apt install spawn-fcgi php php-cgi php-cli \
 apache2 libapache2-mod-fcgid -y

```

<p>Но перед этим необходимо создать файл с настройками для будущего сервиса в файле /etc/spawn-fcgi/fcgi.conf.
Он должен получится следующего вида
</p>

```
root@hwstend:~# nano /etc/spawn-fcgi/fcgi.conf

# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

```

<p>А сам юнит-файл будет примерно следующего вида:</p>

```
root@hwstend:~# nano /etc/systemd/system/spawn-fcgi.service

[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target


```

<p>Убеждаемся, что все успешно работает:</p>

```
root@hwstend:~# systemctl start spawn-fcgi
root@hwstend:~# systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; preset: enabled)
     Active: active (running) since Thu 2025-10-02 13:38:35 UTC; 8s ago
   Main PID: 44571 (php-cgi)
      Tasks: 33 (limit: 4548)
     Memory: 14.6M (peak: 15.0M)
        CPU: 158ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─44571 /usr/bin/php-cgi
             ├─44572 /usr/bin/php-cgi
             ├─44573 /usr/bin/php-cgi
             ├─44574 /usr/bin/php-cgi
             ├─44575 /usr/bin/php-cgi
             ├─44576 /usr/bin/php-cgi
             ├─44577 /usr/bin/php-cgi
             ├─44578 /usr/bin/php-cgi
             ├─44579 /usr/bin/php-cgi
             ├─44580 /usr/bin/php-cgi
             ├─44581 /usr/bin/php-cgi
             ├─44582 /usr/bin/php-cgi
             ├─44583 /usr/bin/php-cgi
             ├─44584 /usr/bin/php-cgi
             ├─44585 /usr/bin/php-cgi
             ├─44586 /usr/bin/php-cgi
             ├─44587 /usr/bin/php-cgi
             ├─44588 /usr/bin/php-cgi
             ├─44589 /usr/bin/php-cgi
             ├─44590 /usr/bin/php-cgi
             ├─44591 /usr/bin/php-cgi
             ├─44592 /usr/bin/php-cgi
             ├─44593 /usr/bin/php-cgi

```

### Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно

<p>Установим Nginx из стандартного репозитория:</p>

```
root@hwstend:~# sudo apt install nginx -y

```

<p>Для запуска нескольких экземпляров сервиса модифицируем исходный service для использования различной конфигурации, а также PID-файлов. Для этого создадим новый Unit для работы с шаблонами (/etc/systemd/system/nginx@.service):
</p>

```
root@hwstend:~# nano /etc/systemd/system/nginx@.service


# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target

```

<p>Далее необходимо создать два файла конфигурации (/etc/nginx/nginx-first.conf, /etc/nginx/nginx-second.conf). Их можно сформировать из стандартного конфига /etc/nginx/nginx.conf, с модификацией путей до PID-файлов и разделением по портам:</p>

```
root@hwstend:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf
root@hwstend:~# nano /etc/nginx/nginx-first.conf
pid /run/nginx-first.pid;

http {
…
	server {
		listen 9001;
	}
#include /etc/nginx/sites-enabled/*;
….
}

root@hwstend:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf
root@hwstend:~# nano /etc/nginx/nginx-second.conf
pid /run/nginx-second.pid;

http {
…
	server {
		listen 9001;
	}
#include /etc/nginx/sites-enabled/*;
….
}

```

<p>Этого достаточно для успешного запуска сервисов.
Проверим работу:
</p>

```
hwuser@hwstend:~$ sudo systemctl start nginx@first
hwuser@hwstend:~$ sudo systemctl start nginx@second
hwuser@hwstend:~$ systemctl status nginx@second.service
● nginx@second.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     Active: active (running) since Tue 2025-10-07 07:57:27 UTC; 14s ago
       Docs: man:nginx(8)
    Process: 2117652 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-second.conf -q -g daemon on; master_process on; (code=exited, s>
    Process: 2117653 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on; (code=exited, status=0/S>
   Main PID: 2117655 (nginx)
      Tasks: 5 (limit: 4548)
     Memory: 4.1M (peak: 4.5M)
        CPU: 64ms
     CGroup: /system.slice/system-nginx.slice/nginx@second.service
             ├─2117655 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;"
             ├─2117656 "nginx: worker process"
             ├─2117657 "nginx: worker process"
             ├─2117658 "nginx: worker process"
             └─2117659 "nginx: worker process"

окт 07 07:57:27 hwstend systemd[1]: Starting nginx@second.service - A high performance web server and a reverse proxy server...
окт 07 07:57:27 hwstend systemd[1]: Started nginx@second.service - A high performance web server and a reverse proxy server.


```
