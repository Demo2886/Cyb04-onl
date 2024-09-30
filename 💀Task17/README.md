# Для выполнения задач, связанных с Docker и проверкой безопасности образа выполним следующие шаги:

### 0. Проверка установлен ли Docker

![img](/Cyb04-onl/💀Task17/img/docker-v.png)

### 1. Скачать образ Ubuntu 18.04 с hub.docker.io

```sh
docker pull ubuntu:18.04
```



### 2. Проверить целостность и соответствие контрольной суммы образа SHA256

Для проверки контрольной суммы образа, выполним следующую команду:

```sh
docker image inspect ubuntu:18.04 --format '{{.RepoDigests}}'
```

Эта команда покажет хеш-сумму SHA256 образа. Вы можете сравнить её с хеш-суммой на Docker Hub.

### 3. Отобразим все Docker образы в системе

```sh
docker image ls
```

### 4. Добавить пользователя в группу Docker

Для запуска команд Docker без sudo, добавьте вашего пользователя в группу Docker:

```sh
sudo usermod -aG docker $USER
```

После этого необходимимо выйдити из терминала и войдите снова, чтобы изменения вступили в силу.

### 5. Запустить образ в интерактивном режиме в оболочке sh

```sh
docker run -it ubuntu:18.04 sh
```

### 6. Выполнить команду whoami внутри контейнера

Внутри контейнера выполните:

```sh
whoami
```

### 7. Запустить контейнер под пользователем tms

Для запуска контейнера под пользователем `tms`, нужно создать этого пользователя внутри контейнера. Можно сделать с помощью Dockerfile или команды `docker run` с опцией `--user`.

Создадим Dockerfile:

```Dockerfile
FROM ubuntu:18.04
RUN useradd -m tms
USER tms
CMD ["sh"]
```

Соберем образ:

```sh
docker build -t ubuntu-tms:18.04 .
```

Запустим контейнер:

```sh
docker run -it ubuntu-tms:18.04
```

### 8. Проверим образ через сканер безопасности Trivy

Установим Trivy:

```sh
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

Просканируем образ:

```sh
trivy image ubuntu:18.04
```

Анализируем результаты сканирования, чтобы убедиться, что образ не содержит уязвимостей.