## SELINUX
### Запустить nginx на нестандартном порту 3-мя разными способами:
<ul>
  <li>1 переключатели setsebool;</li>
  <li>2 добавление нестандартного порта в имеющийся тип;</li>
  <li>3 формирование и установка модуля SELinux.</li>
</ul>

### 1 С помощью переключателя setsebool разрешим в SELinux работу nginx на порту TCP 4881.

<p>С помощью утилиты audit2why смотрми и анализируем в логах (/var/log/audit/audit.log) информацию о блокировании порта </p>

```
[hwuser@localhost ~]$ sudo less /var/log/audit/audit.log | grep "nginx"
[sudo] password for hwuser:
type=ADD_GROUP msg=audit(1761392679.959:404): pid=18847 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:groupadd_t:s0-s0:c0.c1023 msg='op=add-group id=995 exe="/usr/sbin/groupadd" hostname=? addr=? terminal=? res=success'UID="root" AUID="hwuser" ID="nginx"
type=GRP_MGMT msg=audit(1761392679.993:405): pid=18847 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:groupadd_t:s0-s0:c0.c1023 msg='op=add-shadow-group id=995 exe="/usr/sbin/groupadd" hostname=? addr=? terminal=? res=success'UID="root" AUID="hwuser" ID="nginx"
type=ADD_USER msg=audit(1761392680.125:406): pid=18850 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:useradd_t:s0-s0:c0.c1023 msg='op=add-user acct="nginx" exe="/usr/sbin/useradd" hostname=? addr=? terminal=? res=success'UID="root" AUID="hwuser"
type=SOFTWARE_UPDATE msg=audit(1761392681.898:408): pid=18836 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-filesystem-2:1.26.3-1.el10.noarch" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="dnf" exe="/usr/bin/python3.12" hostname=localhost.localdomain addr=? terminal=pts/1 res=success'UID="root" AUID="hwuser"
type=SOFTWARE_UPDATE msg=audit(1761392681.898:409): pid=18836 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-core-2:1.26.3-1.el10.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="dnf" exe="/usr/bin/python3.12" hostname=localhost.localdomain addr=? terminal=pts/1 res=success'UID="root" AUID="hwuser"
type=SOFTWARE_UPDATE msg=audit(1761392681.899:411): pid=18836 uid=0 auid=1000 ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw="nginx-2:1.26.3-1.el10.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="dnf" exe="/usr/bin/python3.12" hostname=localhost.localdomain addr=? terminal=pts/1 res=success'UID="root" AUID="hwuser"
type=AVC msg=audit(1761392886.472:538): avc:  denied  { name_bind } for  pid=19169 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1761392886.472:538): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55adc915bf38 a2=10 a3=7ffebc4a1c30 items=0 ppid=1 pid=19169 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1761392886.477:539): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
type=AVC msg=audit(1761397653.771:567): avc:  denied  { name_bind } for  pid=19358 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1761397653.771:567): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55fba238ef38 a2=10 a3=7ffc6e366980 items=0 ppid=1 pid=19358 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1761397653.779:568): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
type=AVC msg=audit(1761398500.482:680): avc:  denied  { name_bind } for  pid=19502 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
type=SYSCALL msg=audit(1761398500.482:680): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55e94a2abf38 a2=10 a3=7ffd434eca20 items=0 ppid=1 pid=19502 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1761398500.489:681): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
[hwuser@localhost ~]$ sudo grep 1761398500.482:680 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1761398500.482:680): avc:  denied  { name_bind } for  pid=19502 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

	Was caused by:
	The boolean nis_enabled was set incorrectly.
	Description:
	Allow nis to enabled

	Allow access by executing:
	# setsebool -P nis_enabled 1
```
<p>Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр nis_enabled. 
Включим параметр nis_enabled и перезапустим nginx: setsebool -P nis_enabled on
</p>

```
[hwuser@localhost ~]$ sudo setsebool -P nis_enabled 1
[hwuser@localhost ~]$ sudo systemctl start nginx
[hwuser@localhost ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sat 2025-10-25 16:44:38 MSK; 54s ago
 Invocation: 4319f61891134966b5b9d14f560c0410
    Process: 4573 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 4575 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 4576 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 4578 (nginx)
      Tasks: 3 (limit: 10698)
     Memory: 4.2M (peak: 4.4M)
        CPU: 154ms
     CGroup: /system.slice/nginx.service
             ├─4578 "nginx: master process /usr/sbin/nginx"
             ├─4579 "nginx: worker process"
             └─4580 "nginx: worker process"

Oct 25 16:44:38 localhost.localdomain systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Oct 25 16:44:38 localhost.localdomain nginx[4575]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Oct 25 16:44:38 localhost.localdomain nginx[4575]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Oct 25 16:44:38 localhost.localdomain systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
[hwuser@localhost ~]$ sudo systemctl stop nginx
[hwuser@localhost ~]$ sudo setsebool -P nis_enabled off
```

### 2 С помощью добавления нестандартного порта в имеющийся тип

<p>Добавим порт 4881 в тип http_port_t: </p>

```
[hwuser@localhost ~]$ sudo semanage port -a -t http_port_t -p tcp 4881
[hwuser@localhost ~]$ semanage port -l | grep  http_port_t
ValueError: SELinux policy is not managed or store cannot be accessed.
[hwuser@localhost ~]$ sudo semanage port -l | grep  http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[hwuser@localhost ~]$ sudo systemctl start nginx
[hwuser@localhost ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sat 2025-10-25 16:54:36 MSK; 9s ago
 Invocation: 5a9ae6cec3654c6a9cd3cb5cdc8d92ce
    Process: 4662 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 4664 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 4665 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 4667 (nginx)
      Tasks: 3 (limit: 10698)
     Memory: 2.8M (peak: 3.2M)
        CPU: 153ms
     CGroup: /system.slice/nginx.service
             ├─4667 "nginx: master process /usr/sbin/nginx"
             ├─4668 "nginx: worker process"
             └─4669 "nginx: worker process"

Oct 25 16:54:36 localhost.localdomain systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Oct 25 16:54:36 localhost.localdomain nginx[4664]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Oct 25 16:54:36 localhost.localdomain nginx[4664]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Oct 25 16:54:36 localhost.localdomain systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
hwuser@localhost ~]$ sudo semanage port -d -t http_port_t -p tcp 4881
[hwuser@localhost ~]$ sudo systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```


### 3 формирование и установка модуля SELinux

<p>Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux</p>
<p>Посмотрим логи SELinux, которые относятся к Nginx и натравим на них утилиту audit2allow </p>

```
[hwuser@localhost ~]$ sudo grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
```

<p>Audit2allow сформировал модуль, и сообщил нам команду, с помощью которой можно применить данный модуль и проверить работу nginx</p>

```
[hwuser@localhost ~]$ sudo semodule -i nginx.pp
[hwuser@localhost ~]$ sudo systemctl start nginx
[hwuser@localhost ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sat 2025-10-25 17:05:32 MSK; 9s ago
 Invocation: 6c6764ee230f4e059d0ec8c2d56371cb
    Process: 4773 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 4775 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 4777 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 4778 (nginx)
      Tasks: 3 (limit: 10698)
     Memory: 2.8M (peak: 3.1M)
        CPU: 150ms
     CGroup: /system.slice/nginx.service
             ├─4778 "nginx: master process /usr/sbin/nginx"
             ├─4779 "nginx: worker process"
             └─4780 "nginx: worker process"

Oct 25 17:05:32 localhost.localdomain systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
Oct 25 17:05:32 localhost.localdomain nginx[4775]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Oct 25 17:05:32 localhost.localdomain nginx[4775]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Oct 25 17:05:32 localhost.localdomain systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
```
