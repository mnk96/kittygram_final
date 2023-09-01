#  Контейнеры и CI/CD для Kittygram

## Описание проекта
Социальная сеть для обмена фотографиями любимых питомцев. Проект доступен по адресу: https://nataska-kittygram.hopto.org
____
![example workflow](https://github.com/mnk96/kittygram_final/actions/workflows/main.yml/badge.svg)

## Стек технологий:
____
* Python 3.10.12
* Node.js
* Django
* React
* Gunicorn
* Ngnix
* Docker

## Как запустить проект:
____
Команды указаны для Linux.
1. Клонировать репозиторий и перейти в него в командной строке:
```sh
git clone git@github.com:mnk96/infra_sprint1.git
```
2. Активировать виртуальное окружение.
```sh
# Создаём виртуальное окружение.
python3 -m venv venv
# Активируем виртуальное окружение.
source venv/bin/activate
```
3. Установить зависимости
```sh
pip install -r requirements.txt 
```
4. Выполнить миграции
```sh
python manage.py migrate
```
5. Создать суперпользователя
```sh
python manage.py createsuperuser 
```

## Создание Docker_образов
1. Замените username на ваш логин на DockerHub:
```sh
cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway . 
```
2. Загрузите образы на DockerHub:
```sh
docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway
```

## Деплой на сервер

1. Подключитесь к удаленному серверу
```sh
ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
```

2. Остановите Gunicorn
```sh
sudo systemctl stop gunicorn_kittygram
```

3. Удалите юнит gunicorn
```sh
sudo rm /etc/systemd/system/gunicorn_kittygram.service 
```

4. Установите Docker Compose на сервер
```sh
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin 
```

5. Скопируйте файл docker-compose.production.yml. в директорию infra_sprint1/
```sh
# 1 вариант
scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/taski/docker-compose.production.yml
# 2 вариант
cd infra_sprint1
sudo nano docker-compose.production.yml
# Скопировать в файл содержимое docker-compose.production.yml на GitHub
```

6. Скопируйте файл .env на сервер, в директорию infra_sprint1/
```sh
sudo nano .env
```
Содержимое .env должно соответствовать примеру:
```sh
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
DB_HOST=db
DB_PORT=5432
```

7. Запустите Docker Compose в режиме демона в папке infra_sprint1/
```sh
sudo docker compose -f docker-compose.production.yml up -d 
```

8. Проверьте наличие запущенных контейнеров
```sh
sudo docker compose -f docker-compose.production.yml ps
```

9. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/
```sh
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

10. Перенаправьте запросы в Docker
```sh
sudo nano /etc/nginx/sites-enabled/default
```

11. Измените содердимое файла
```sh
# Всё до этой строки оставляем как было.
    location / {
        proxy_pass http://127.0.0.1:9000;
    }
# Ниже ничего менять не нужно.
```

12. Проверьте конфиг на правильность
```sh
sudo nginx -t 
```

13. Перезагрузите конфиг Nginx
```sh
sudo service nginx reload 
```

## Автоматизация деплоя: CI/CD

1. Создайте в папке kittygram_final/ директорию .github/workflows, а в ней — файл main.yml

2. Скопируйте в этой файл содержимое файла kittygram_workflow.yml

3. Создайте секреты в GitHub Actions
```sh
DOCKER_USERNAME - логин для Docker Hub
DOCKER_PASSWORD - пароль для Docker Hub
SSH_KEY - закрытый SSH-ключ для доступа к серверу
SSH_PASSPHRASE - passphrase для этого ключа
USER - имя пользователя
HOST - IP-адрес вашего сервера
TELEGRAM_TO - ID своего телеграм-аккаунта
TELEGRAM_TOKEN - токен вашего бота
```

3. Запустите workflow
```sh
Dev/kittygram_final$ git add .
Dev/kittygram_final$ git commit -m 'Add Actions'
Dev/kittygram_final$ git push 
```
## Автор проекта:
Мищенко Наталья