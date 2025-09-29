## Cборка пакета Nginx с модулем ngx_brotli и создание своего apt-репозитория
<p>Устанавливаем все необходимые пакеты для сборки:</p>

```bash
hwuser@hwsrv:~$ sudo apt update
hwuser@hwsrv:~$ sudo apt install -y wget git nano build-essential devscripts \
    fakeroot lintian cmake dpkg-dev
```
<p>Скачиваем исходные коды пакета и зависимости для сборки:</p>

```
hwuser@hwsrv:~$ mkdir ~/deb && cd ~/deb
hwuser@hwsrv:~$ sudo apt source nginx
hwuser@hwsrv:~$ sudo apt build-dep nginx -y
```
<p>Клонируем модуль и собираем библиотеку Brotli:</p>

```
root@hwsrv:/home/hwuser/deb# cd ~
root@hwsrv:~# git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
root@hwsrv:~# cd ngx_brotli/deps/brotli
root@hwsrv:~/ngx_brotli/deps/brotli# mkdir out && cd out
root@hwsrv:~/ngx_brotli/deps/brotli/out# cmake -DCMAKE_BUILD_TYPE=Release ..
root@hwsrv:~/ngx_brotli/deps/brotli/out# cmake --build . --config Release -j2 --target brotlienc

root@hwsrv:~/ngx_brotli/deps/brotli/out#  cd ~
```
<p>Редактируем правила сборки Nginx, чтобы добавить поддержку модуля:</p>

```
root@hwsrv:~# cd /home/veles/deb/nginx-*/
root@hwsrv:/home/veles/deb/nginx-1.24.0# nano debian/rules
```
<p>В секции common_configure_flags  (там где --with-cc-opt и --with-ld-opt) добавляем строку:</p>

```
--add-module=/home/hwuser/ngx_brotli \
```
<p>Запускаем сборку DEB пакета</p>

```
debuild -us -uc -b
........
Now running lintian nginx_1.24.0-2ubuntu7.5_amd64.changes ...
running with root privileges is not recommended!
Finished running lintian.
```


```
hwuser@hwsrv:~/deb$ sudo dpkg -i ~/deb/nginx_1.24.0-2ubuntu7.5_amd64.deb
hwuser@hwsrv:~/deb$ sudo systemctl status nginx.service
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-09-29 16:07:07 UTC; 45s ago
       Docs: man:nginx(8)
    Process: 33300 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 33301 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 33602 (nginx)
      Tasks: 5 (limit: 2210)
     Memory: 3.7M (peak: 8.2M)
        CPU: 90ms
     CGroup: /system.slice/nginx.service
             ├─33602 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─33605 "nginx: worker process"
             ├─33606 "nginx: worker process"
             ├─33607 "nginx: worker process"
             └─33608 "nginx: worker process"

сен 29 16:07:07 hwsrv systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
сен 29 16:07:07 hwsrv systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.

```

## Создание своего APT-репозитория
<p>Копируем пакеты в директорию Nginx:</p>

```
hwuser@hwsrv:~/deb$ sudo mkdir -p /usr/share/nginx/html/repo
hwuser@hwsrv:~/deb$ sudo cp ~/deb/*.deb /usr/share/nginx/html/repo/

```
<p>Создаем индекс пакетов:</p>

```
root@hwsrv:~# cd /usr/share/nginx/html/repo
root@hwsrv:/usr/share/nginx/html/repo# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
dpkg-scanpackages: warning: Packages in archive but missing from override file:
dpkg-scanpackages: warning:   libnginx-mod-http-geoip libnginx-mod-http-image-filter libnginx-mod-http-perl libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip nginx nginx-common nginx-core nginx-dev nginx-doc nginx-extras nginx-full nginx-light
dpkg-scanpackages: info: Wrote 15 entries to output Packages file.

```
### Настройка доступа через Nginx
<p>Редактируем конфигурацию:</p>

```
root@hwsrv:/usr/share/nginx/html/repo# nano /etc/nginx/sites-available/default
```
<p>Проверяем конфигурацию и перезапускаем:</p>

```
hwuser@hwsrv:/usr/share/nginx/html/repo$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
hwuser@hwsrv:/usr/share/nginx/html/repo$ sudo systemctl restart nginx.service
```

### Подключение репозитория к APT

<p>Добавляем локальный репозиторий:</p>

```
hwuser@hwsrv:/usr/share/nginx/html/repo$ sudo echo "deb [trusted=yes] http://localhost/repo ./" | sudo tee /etc/apt/sources.list.d/localrepo.list
hwuser@hwsrv:/usr/share/nginx/html/repo$ sudo apt update
```
### Проверка работы

```
hwuser@hwsrv:/usr/share/nginx/html/repo$ sudo apt list -a | grep nginx
nginx/noble-updates,noble-security,noble-updates,noble-security,now 1.24.0-2ubuntu7.5 amd64 [installed,local]
```
