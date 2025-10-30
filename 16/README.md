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
veles@ansible:~$ mkdir -p ~/ansible/roles/web/roles/nginx/{tasks,templates,handlers,vars,defaults,files,meta}
veles@ansible:~$ tree ./ansible/
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
192.168.56.10 ansible_user=veles ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 8 Запускам Playbook

```
ansible-playbook -i inventory site.yml
```

### 9 Проверям на подчиненной виртуальной машине

```
sudo systemctl status nginx
ss -tuln | grep 8080
curl http://localhost:8080
```
