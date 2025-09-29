## Cборка пакета Nginx с модулем ngx_brotli и создание своего apt-репозитория
<p>Устанавливаем все необходимые пакеты для сборки:</p>
``` bash
sudo apt update
sudo apt install -y wget git nano build-essential devscripts \
    fakeroot lintian cmake dpkg-dev
```
