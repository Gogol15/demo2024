# DOCKER
Устанавливам машину на базе ОС ALT Linux.

Обновляем все пакеты:
```
apt-get update
```
Устанавливаем пакет Docker:
```
apt-get install -y docker-engine
```
Запускаем Docker:
```
systemctl enable --now docker
```
Скачиваем контейнер Hello-World:
```
docker pull hello-world
```
Смотрим список контейнеров:
```
docker images
```
Запускаем контейнер Hello-World:
```
docker run hello-world
```
Останавливаем контейнер:
```
docker rm -v *ID контейнера* 
```
Удаляем контейнер:
```
docker rmi *ID контейнера*
```
