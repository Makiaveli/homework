# Docker: основы работы с контейнеризацией
## Установка Docker 
### Добавляем репозиторий

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```
### Устнавливаем Docker 
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
#### P/s в последних версиях модуль docker compose отдельно устанавливать не нужно он уже присутсвует в репозитории.

### Добавляем пользователя в группу docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```
## Собираем образ
```bash
docker build -t velestr/nginx-marvel:1.0 .
```
## Запускаем кониейнер 
```bash
docker run -d -p 8080:80 --name nginx-marvel velestr/nginx-marvel:1.0
```

## Разница между образом и контейнером 

### Docker-образ — это:
<ul>
  <li>неизменяемый шаблон</li>
  <li>набор слоёв (read-only)</li>
  <li>инструкция как создать контейнер</li>
</ul>

### Docker-контейнер — это:
<ul>
  <li>запущенный экземпляр образа</li>
  <li>изолированный процесс</li>
  <li>имеет writable-layer</li>
</ul>

## Можно ли в контейнере собрать ядро Linux?
<p> ДА, но с ограничениями.</p>
<ul>
  <li>В контейнере можно скомпилировать ядро</li>
  <li>Но контейнер НЕ использует своё ядро</li>
  <li>Контейнер всегда работает на ядре хост-системы</li>
</ul>

## Пуш образа в Docker Hub

```bash
docker push velestr  velestr/nginx-marvel:1.0
```

## Ссылка на образ <a href="https://hub.docker.com/r/velestr/nginx-marvel">dockerhub</a>


# Задание со звездочкой
<ul>
  <li>Написать <a href ="https://github.com/Makiaveli/homework/blob/main/19/docker-compose.yml">Docker-compose</a> для приложения Redmine, с использованием опции build.</li>
  <li>Добавить в базовый образ redmine любую кастомную тему оформления.</li>
  <li>Убедиться что после сборки новая тема доступна в настройках.</li>
  <li>Настроить вольюмы, для сохранения всей необходимой информации
</li>
</ul>

