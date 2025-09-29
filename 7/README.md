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
cd ~
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release -j2 --target brotlienc
cd ../../../..
```
