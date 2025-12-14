# Docker: основы работы с контейнеризацией
## Установка Docker 
### Добавляем репозиторий

```
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
#### P/s в последних версиях модуль docker compose отдельно устанавливать не нужно он уже присутсвует в репозитории.

### Добавляем пользователя в группу docker

```
sudo usermod -aG docker $USER
newgrp docker
```
