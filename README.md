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

cp -r .env compose.yaml proxy.yaml nginx haproxy /runtimedir

cd ..
rm -rf ${GH_REPO}-${GH_BRANCH}

#Запускаем проект
cd /runtimedir
docker compose up -d
```
### Ссылка на репозиторий
https://github.com/pkostua/shvirtd-example-python


