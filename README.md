# 05-virt-04-docker-in-practice
Решение домашнего задания к занятию 5. «Практическое применение Docker»
https://github.com/netology-code/virtd-homeworks/blob/shvirtd-1/05-virt-04-docker-in-practice/README.md
## Задача 2
![image](https://github.com/user-attachments/assets/2cf7a25f-df2a-48a7-a701-a6e04d9350f3)
## Задача 3
Смотрю - в задании ненадо скриншоты консоли делать, поэтому решил все сделать на windows. Но там, как оказалось из коробки не работает network_mode: host. Пришлось дебажить, делать запросы на 8080
![image](https://github.com/user-attachments/assets/29256f5f-dc10-435a-9df5-3f16b73eceda)
## Задача 4
### Скриншот  SQL запроса. 
Приложение смогло осилить только два запроса 
![image](https://github.com/user-attachments/assets/f90272c0-e78a-4a17-8fba-20c969779e5c)
### Билд деплой скрипт проекта
```
#Объявляем переменные
GH_USER=pkostua
GH_REPO=shvirtd-example-python
GH_BRANCH=main

#Скачиваем код
wget "https://codeload.github.com/${GH_USER}/${GH_REPO}/tar.gz/refs/heads/${GH_BRANCH}" -O "${GH_REPO}-${GH_BRANCH}.tar.gz"
tar -xzvf ./"${GH_REPO}-${GH_BRANCH}.tar.gz"
rm ./"${GH_REPO}-${GH_BRANCH}.tar.gz"

#Собираем приложение
cd ${GH_REPO}-${GH_BRANCH}
docker build --tag ${GH_REPO}:${GH_BRANCH} -f Dockerfile.python .

#Копируем все необходимое для запуска
cp -r .env compose.yaml proxy.yaml nginx haproxy /runtimedir

#Удаляем скачанный репозиторий, он не нужен для выполнения
cd ..
rm -rf ${GH_REPO}-${GH_BRANCH}

#Запускаем проект
cd /runtimedir
docker compose up -d
```
### Ссылка на репозиторий
https://github.com/pkostua/shvirtd-example-python

## Задание 5.
Мне не удалось выполнить эту таску в точном соответствии с текстом задания. При запуске schnitzler/mysqldump возникала ошибка по отсутствию библиотеки caching_sha2_password.so.  Ниже будет предствалено один из вариантов решения задания.
Для обхода ошибки сделаем новый образ на основе schnitzler/mysqldump, в котрый добавим недостающие библиотеки.
### Dockerfile
```
FROM schnitzler/mysqldump
RUN apk add --no-cache mysql-client mariadb-connector-c
```
Собираем образ, и запускам скрипт с использованием нового образа
## Скрипт бакапа
```
# Загрузка переменных из .env файла
set -a
source /runtimedir/.env
set +a

BACKUP_DIR="/opt/backup"
TIMESTAMP=$(date +"%Y%m%d%H%M")

# Создание директории для резервных копий, если она не существует
mkdir -p "$BACKUP_DIR"

# Выполнение резервного копирования с помощью Docker
docker run \
    --network runtimedir_backend\
    --rm --entrypoint "" \
    -v $BACKUP_DIR:/backup \
    --link="container:runtimedir-db-1" \
    cr.yandex\crpj0at1oiaj33rl71e0\mysqldump \
    mysqldump --opt -h "db"  -uroot -p"$MYSQL_ROOT_PASSWORD"  "--result-file=/backup/$MYSQL_DATABASE-$TIMESTAMP.sql" "$MYSQL_DATABASE"

# Удаление старых резервных копий (например, старше 7 дней)
find "$BACKUP_DIR" -type f -name "*.sql" -mtime +7 -exec rm {} \;
```
## Список бакапов
![image](https://github.com/user-attachments/assets/b20b2c43-c307-4d11-bbfb-90c1df102750)
## crontab -e
