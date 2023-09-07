# taski-docker
### Описание проекта 
taski - сервис для любителей котиков.

Что умеет проект:

- Добавлять, просматривать, редактировать и удалять котиков.
- Добавлять новые и присваивать уже существующие достижения. 
- Просматривать чужих котов и их достижения.

## Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```bash
    git clone https://github.com/klassnenkiy/taski-docker.git
    ```
    ```bash
    cd taski_final
    ```
2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```bash
    cd backend
    docker build -t tyulyagin/taski_backend .
    cd frontend
    docker build -t tyulyagin/taski_frontend .
    cd gateway
    docker build -t tyulyagin/taski_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push tyulyagin/taski_backend
    docker push tyulyagin/taski_frontend
    docker push tyulyagin/taski_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i /home/ea703557/Загрузки/555/yc-ea703557 yc-user@158.160.28.33
key NRjeSf
 имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию taski через терминал

    ```bash
    mkdir taski
    ```

3. Установка docker compose на сервер:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию taski/ скопируйте файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/taski/docker-compose.production.yml
    * ath_to_SSH — путь к файлу с SSH-ключом;
    * SSH_name — имя файла с SSH-ключом (без расширения);
    * username — ваше имя пользователя на сервере;
    * server_ip — IP вашего сервера.g

 scp -i /home/ea703557/Загрузки/555/yc-ea703557 docker-compose.production.yml  yc-user@158.160.28.33:/home/yc-user/taski/docker-compose.production.yml

 scp -i /home/ea703557/Загрузки/555/yc-ea703557 .env  yc-user@158.160.28.33:/home/yc-user/taski/.env
 key NRjeSf

 Добавьте в файл .env переменные и их значения:
# Файл .env
P# Файл .env
POSTGRES_USER=django_user
POSTGRES_PASSWORD=mysecretpassword
POSTGRES_DB=django
# Добавляем переменные для Django-проекта:
DB_HOST=db
DB_PORT=5432

 
POSTGRES_USER — имя пользователя БД (необязательная переменная, значение по умолчанию — postgres);
POSTGRES_PASSWORD — пароль пользователя БД (обязательная переменная для создания БД в контейнере);
POSTGRES_DB — название базы данных (необязательная переменная, по умолчанию совпадает с POSTGRES_USER).
Таким образом, можно передать в окружение только переменную POSTGRES_PASSWORD — и будет создана БД с названием postgres и пользователем postgres.
При запуске контейнера имя файла с переменными окружения передают в параметр --env-file:
# Старт контейнера с именем db, переменные окружения в .env, 
# контейнер запустить из образа postgres:13.10
# Эта команда – для демонстрации, запускать ее не надо
docker run --name db --env-file .env postgres:13.10 
    ```

    docker run -d \
	--name some-postgres \
	-e POSTGRES_PASSWORD=mysecretpassword \
	-e PGDATA=/var/lib/postgresql/data/pgdata \
	-v /custom/mount:/var/lib/postgresql/data \
	postgres

sudo docker run --name db \
       --env-file .env \
       -v pg_data:/var/lib/postgresql/data \
       postgres:13.10


6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
Далее выполняем последовательно
sudo docker compose -f docker-compose.production.yml pull
sudo docker compose -f docker-compose.production.yml down
sudo docker compose -f docker-compose.production.yml up -d
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collect_static/. /static_backend/static/

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:

    ```bash
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте работоспособность конфига Nginx:

    ```bash
    sudo nginx -t
    ```
    Если ответ в терминале такой, значит, ошибок нет:
    ```bash
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```
10. Перезапускаем Nginx
    ```bash
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории

    ```bash
    taski/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # ip_address сервера
    USER                           # имя пользователя
    SSH_KEY                        # приватный ssh-ключ (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # кодовая фраза (пароль) для ssh-ключа
    TELEGRAM_TO                    # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```


###  Как работать с репозиторием финального задания

#### Что нужно сделать

Настроить запуск проекта taski в контейнерах и CI/CD с помощью GitHub Actions

#### Как проверить работу с помощью автотестов

В корне репозитория создайте файл tests.yml со следующим содержимым:
```yaml
repo_owner: ваш_логин_на_гитхабе
taski_domain: полная ссылка (https://доменное_имя) на ваш проект taski
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```

Скопируйте содержимое файла `.github/workflows/main.yml` в файл `taski_workflow.yml` в корневой директории проекта.

Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из backend/requirements.txt и запустите в корневой директории проекта `pytest`.

#### Чек-лист для проверки перед отправкой задания

- Проект Taski доступен по доменному имени, указанному в `tests.yml`.
- Проект taski доступен по доменному имени, указанному в `tests.yml`.
- Пуш в ветку main запускает тестирование и деплой taski, а после успешного деплоя вам приходит сообщение в телеграм.
- В корне проекта есть файл `taski_workflow.yml`.



```sudo nano /etc/nginx/sites-enabled/default```:

```
server {
    server_name 158.160.28.33 klassnenkiy.ddns.net;
    server_tokens off;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:8000;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/klassnenkiy.ddns.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/klassnenkiy.ddns.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    server_name 158.160.28.33 klassnenkiy.ddns.net;
    server_tokens off;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/klassnenkiy.ddns.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/klassnenkiy.ddns.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = klassnenkiy.ddns.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name 158.160.28.33 klassnenkiy.ddns.net;
    return 404; # managed by Certbot
}

server {
    if ($host = klassnenkiy.ddns.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name 158.160.28.33 klassnenkiy.ddns.net;
    return 404; # managed by Certbot
}
```

On your remote server in the root directory of the project you should also place the ```docker-compose.production.yml``` :


```
version: '3'

volumes:
  pg_data_production:
  static_volume:

services:
  db:
    image: postgres:13.10
    env_file: .env
    volumes:
      - pg_data_production:/var/lib/postgresql/data
  backend:
    image: tyulyagin/taski_backend # Docker Hub username / username on Docker image
    env_file: .env
    volumes:
      - static_volume:/backend_static
  frontend:
    image: tyulyagin/taski_frontend  # Download from Docker Hub
    env_file: .env
    command: cp -r /app/build/. /frontend_static/
    volumes:
      - static_volume:/frontend_static
  gateway:
    image: tyulyagin/taski_gateway  # Download from с Docker Hub
    env_file: .env
    volumes:
      - static_volume:/staticfiles/
    ports:
      - 8000:80
```

 
