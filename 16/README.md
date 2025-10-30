## Автоматизация администрирования. Ansible-1 *
<p>На сервере, используя Ansible, необходимо развернуть nginx со следующими условиями:</p>
<ul>
  <li>необходимо использовать модуль yum/apt;</li>
  <li>конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными;</li>
  <li>после установки nginx должен быть в режиме enabled в systemd;</li>
  <li>должен быть использован notify для старта nginx после установки;</li>
  <li>сайт должен слушать на нестандартном порту — 8080, для этого использовать переменные в Ansible.</li>
</ul>

### 1 на управляющем хосте создаем структуру для ansible роли

```
hwuser@ansible:~$ mkdir -p ~/ansible/roles/nginx/{tasks,templates,handlers,vars,defaults,files,meta}
hwuser@ansible:~$ tree ./ansible/
./ansible/
└── roles
    └── web
        └── roles
            └── nginx
                ├── defaults
                ├── files
                ├── handlers
                ├── meta
                ├── tasks
                ├── templates
                └── vars

12 directories, 0 files

```
### 2 Определяем переменные (vars/main.yml)

```
---
nginx_port: 8080
nginx_user: www-data
nginx_worker_processes: auto
```

### 3 Определяем задачи (tasks/main.yml)

```
---
- name: Установка Nginx
  become: true
  ansible.builtin.package:
    name: nginx
    state: present
  notify: Запустить nginx

- name: Создание конфигурационного файла из шаблона
  become: true
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Перезапустить nginx

- name: Включить nginx в автозагрузку
  become: true
  ansible.builtin.systemd:
    name: nginx
    enabled: true
```

### 4 Определяем обработчики (handlers/main.yml)

```
---
- name: Запустить nginx
  become: true
  ansible.builtin.systemd:
    name: nginx
    state: started

- name: Перезапустить nginx
  become: true
  ansible.builtin.systemd:
    name: nginx
    state: restarted
```

### 5 Созаздаем шаблон (templates/nginx.conf.j2)

```
user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout 65;

    server {
        listen {{ nginx_port }};
        server_name localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}

```
### 6 Создаем Playbook для вызова роли

```
---
- name: Развертывание nginx через роль
  hosts: web
  become: true
  roles:
    - nginx
```

### 7 Создаем inventory файл 

```
[web]
192.168.56.10 ansible_user=hwuser ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 8 Запускам Playbook

```
hwuser@ansible:~/ansible$ ansible-playbook -i inventory.ini site.yml

```

### 9 Проверям на подчиненной виртуальной машине

```
hwuser@ansible:~/ansible$ sudo systemctl status nginx
nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-10-30 11:49:17 UTC; 2min 51s ago
       Docs: man:nginx(8)
    Process: 24358 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 24361 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 24363 (nginx)
      Tasks: 5 (limit: 2210)
     Memory: 4.1M (peak: 4.3M)
        CPU: 39ms
     CGroup: /system.slice/nginx.service
             ├─24363 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─24364 "nginx: worker process"
             ├─24365 "nginx: worker process"
             ├─24366 "nginx: worker process"
             └─24367 "nginx: worker process"

окт 30 11:49:17 hwsrv systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
окт 30 11:49:17 hwsrv systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

```
hwuser@ansible:~/ansible$ ss -tuln | grep 8080
tcp   LISTEN 0      511          0.0.0.0:8080      0.0.0.0:*
```

```
hwuser@ansible:~/ansible$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
