#  Проект Taski для задач

## Описание

Проект для добавления, редактирования и удаления поставленных задач пользователя

Taski в контейнерах, и CI/CD с помощью GitHub Actions

## Развернутый проект

```
https://letsgo.sytes.net/
```

[![Main Taski workflow](https://github.com/DonBenn/taski-docker/actions/workflows/main.yml/badge.svg)](https://github.com/DonBenn/taski-docker/actions/workflows/main.yml)

## Версии запуска

В проекте представленны две версии запуска проекта. Первая версия описанна в файле 
docker-compose.yml , в ней новые образы билдятся при каждом запуске, это удобно для отладки.
Вторая версия представленна в файле docker-compose.production.yml . В этом конфиге запускаются уже готовые образы для контейнеров, которые не билдятся, а скачиваются с Docker Hub. Это используется в продакшене.

## Локальный запуск проекта

Клонировать репозиторий и перейти в него в командной строке:

```
git clone https://github.com/DonBenn/taski-docker.git
```

```
cd backend
```

Cоздать и активировать виртуальное окружение:

```
python -m venv venv
```

```
source venv/Scripts/activate
```

Установить зависимости из файла requirements.txt:

```
python -m pip install --upgrade pip
```

```
pip install -r requirements.txt
```

Перейдите в директорию, где лежит файл docker-compose.yml и выполните команды:

Сборка образов и запуск контейнеров

```
docker compose up 
```

Выполнение миграций 

```
docker compose exec backend python manage.py migrate
```

Собрать статику Django


```
docker compose exec backend python manage.py collectstatic
```

Копируем статику в volume в папку /backend_static/static/

```
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```

Запуск проекта в браузере

```
http://localhost:9000/
```


Перезапуск и пересборка образов

```
docker compose stop && docker compose up --build 
```

## Удаленное развертывание проекта на сервере


Клонировать репозиторий и перейти в него в командной строке:

```
git clone https://github.com/DonBenn/taski-docker.git
```

Перейдите в директории для сборки образов и выполните команды:

В терминале в корне проекта Taski последовательно выполните команды из листинга; 
замените username на ваш логин на Docker Hub

```
cd frontend  
docker build -t username/taski_frontend . 
cd ../backend  
docker build -t username/taski_backend .
cd ../nginx  
docker build -t username/taski_gateway . 

```

Отправьте собранные образы фронтенда, бэкенда и Nginx на Docker Hub

```
docker push username/taski_frontend
docker push username/taski_backend
docker push username/taski_gateway
```

Запустите Docker Compose с этой конфигурацией на своём компьютере

```
docker compose -f docker-compose.production.yml up 
```

Примените миграции

```
docker compose -f docker-compose.production.yml exec backend python manage.py migrate
```

Подключитесь к удаленному серверу

Переместитесь в домашнюю директорию

```
cd
```

Устанавите Docker Compose на сервер, поочередно выполнив команды:

```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt install docker-compose-plugin 
```

Запускаем Docker Compose на сервере

Перейдите в директорию `taski/`

Cоздайте на сервере пустой файл `docker-compose.production.yml` и с помощью 
редактора nano добавьте в него содержимое из локального `docker-compose.production.yml`

Скопируйте файл .env на сервер, в директорию `taski/`

Запустите Docker Compose в режиме демона

```
sudo docker compose -f docker-compose.production.yml up -d 
```

Проверьте, что все нужные контейнеры запущены:

```
sudo docker compose -f docker-compose.production.yml ps 
```

Выполните миграции, соберите статические файлы бэкенда и скопируйте их 
в `/backend_static/static/`

```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/ 
```

На сервере в редакторе nano откройте конфиг Nginx:

```
nano /etc/nginx/sites-enabled/default
```

Измените настройки location в секции server

```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
```

Сохраните изменения 

```
sudo nginx -t 
```

Перезагрузите конфиг Nginx 

```
sudo service nginx reload
```

Откройте в браузере страницу вашего проекта — https://ваш_домен/



## Используемые технологии

В проекте используются фреймворки Django, Django rest framework, React, Node js.
Для работы с контейнерами используется Docker Desktop.
На сервере используется веб-сервер Nginx.
Также используется CI/CD Github Actions Workflow.


## Автор проекта

```
Бессонов Денис
```

