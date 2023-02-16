Скачать бинарный файл *gitea*, сделать его исполняемыи и поместить в каталог */usr/local/bin* (либо сделать на него символьную ссылку `ln -s gitea /usr/local/bin/gitea`)
```
VERSION=1.18.3
wget -O gitea https://dl.gitea.com/gitea/$VERSION/gitea-$VERSION-linux-amd64
chmod +x gitea
sudo mv gitea /usr/local/bin
```
Добавить пользователя, под которым будет запускаться gitea
```
adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
```
Cоздать структуру каталогов
```
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
```
Создать каталог для настроек
```
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea         ; 770 временно, чтобы пользователь git смог записать файл настроек
```
Cкачать файл gitea.service и отредактировать его (change user, home directory, choose database params and other required startup values + change the `PORT` or remove the `-p` flag if default port is used), после чего запустить сервис
```
sudo wget https://raw.githubusercontent.com/go-gitea/gitea/master/contrib/systemd/gitea.service -P /etc/systemd/system/
sudo vim /etc/systemd/system/gitea.service
sudo systemctl daemon-reload
sudo systemctl enable --now gitea
```
По умолчанию gitea прослушивает соединения на порту `3000` всех сетевых интерфейсов, поэтому необходимо в брандмауэре открыть `3000/tcp` командой `sudo ufw allow 3000/tcp` в ubuntu, либо в centOS:
```
sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent
firewall-cmd --reload
```
Подготовка СУБД
```
mysql -e 'CREATE DATABASE gitea;'
mysql -e 'GRANT ALL ON gitea.* TO \'gitea\'@\'localhost\' IDENTIFIED BY \'MY_PASSWORD\';'
mysql -e 'FLUSH PRIVILEGES;'
```
Выполняем первый (ручной) запуск для установки
```
su - git
GITEA_WORK_DIR=/var/lib/gitea/ /usr/local/bin/gitea web -c /etc/gitea/app.ini
```
Либо переходим по адресу `https://ip_addr:3000` и производим установку через web-интерфейс.\
*Пример настройки базы данных (sqlite3)*
```
Тип базы данных: SQLite3
Путь: абсолютный путь /var/lib/gitea/data/gitea.db
Общие настройки приложения:

Название сайта: название организации.
Корневой путь к хранилищу: оставить значение по умолчанию /home/git/gitea-repositories
Git LFS Root Path: оставить значение по умолчанию /var/lib/gitea/data/lfs
Запуск от имени пользователя: git
Домен сервера SSH: ввести IP-адрес домена или сервера
Порт SSH: 22 либо любо порт, на котором работает SSH
Порт прослушивания HTTP Gitea: 3000
URL базы Gitea: используем http и IP-адрес домена или сервера
Путь к журналу: оставить значение по умолчанию /var/lib/gitea/log
```
После указания настроек нажимаем кнопку *Install Gitea*. После завершения откроется страница входа,
где нужно будет выбрать ссылку *Sign up now*. Первый зарегистрированный пользователь автоматически
добавляется в группу администраторов.

После завершения установки необходимо изменить разрешения для объектов в каталоге */etc/*
```
chmod 750 /etc/gitea
chmod 640 /etc/gitea/app.ini
```
## Настройка Nginx в качестве прокси-сервера завершения SSL
Вносим изменения в файл конфигурации *nginx.conf* (заменяя домен git.example.ru на нужный и указывая пути к сертификатам):
```
server {
    listen 80;
    server_name git.example.ru;

    include snippets/letsencrypt.conf;
    return 301 https://git.example.ru$request_uri;
}

server {
    listen 443 ssl http2;
    server_name git.example.ru;

    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;

    client_max_body_size 50m;

    # Proxy headers
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    # SSL parameters
    ssl_certificate /etc/letsencrypt/live/git.example.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/git.example.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/git.example.ru/chain.pem;
    include snippets/letsencrypt.conf;
    include snippets/ssl.conf;

    # log files
    access_log /var/log/nginx/git.example.ru.access.log;
    error_log /var/log/nginx/git.example.ru.error.log;

    # Handle / requests
    location / {
       proxy_redirect off;
       proxy_pass http://127.0.0.1:3000;
    }
}
```
Перезапускаем службу nginx
```
sudo systemctl restart nginx
```
В файле конфигурации */etc/gitea/app.ini* изменяем домен gitea и корневой url: 
 ```ini
[server]
DOMAIN           = git.example.ru
ROOT_URL         = https://git.example.ru/
```
Перезапускаем сервис gitea:
```
sudo systemctl restart gitea
```

## Настройка почтовых уведомлений
Чтобы gitea могла отправлять уведомления по электронной почте, нужно установить, напр., *Postfix* и отредактировать файл */etc/gitea/app.ini*:
```
[mailer]
ENABLED = true
HOST    = SMTP_SERVER:SMTP_PORT
FROM    = SENDER_EMAIL
USER    = SMTP_USER
PASSWD  = YOUR_SMTP_PASSWORD
```
Перезапускаем службу
```
sudo systemctl restart gitea
```
Чтобы проверить настройки и отправить тестовое письмо, входим в gitea и переъодим по ссылке *Site Administration —> Configuration —> SMTP Mailer Configuration*.

## Обновление Gitea
Для обновления gitea загружаем и заменяем его бинарный файл.
```
sudo systemctl stop gitea
VERSION=<THE_LATEST_GITEA_VERSION>
wget -O gitea https://dl.gitea.io/gitea/${VERSION}/gitea-${VERSION}-linux-amd64
sudo chmod +x gitea
sudo mv gitea /usr/local/bin
sudo systemctl restart gitea
 ```
 
## Файлы конфигурации
### `/etc/systemd/system/gitea.service`
```
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
Wants=mariadb.service
After=mariadb.service=redis.service
[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
[Install]
WantedBy=multi-user.target
```

### `/etc/gitea/app.ini`
```
APP_NAME = Gitea
RUN_USER = git
RUN_MODE = prod

[security]
# не трогать
# ...

[database]
# выставить корректные данные для подключения к БД
DB_TYPE  = mysql
HOST     = 127.0.0.1:3306
NAME     = gitea
USER     = gitea
PASSWD   = MY_PASSWORD
SCHEMA   = 
SSL_MODE = disable
CHARSET  = utf8
PATH     = /var/lib/gitea/data/gitea.db
LOG_SQL  = false

[repository]
ROOT = /var/lib/gitea/data/gitea-repositories

[server]
SSH_DOMAIN       = localhost
DOMAIN           = localhost
HTTP_PORT        = 8080
ROOT_URL         = # не трогать
DISABLE_SSH      = false
SSH_PORT         = 22
LFS_START_SERVER = true
LFS_CONTENT_PATH = /var/lib/gitea/data/lfs
LFS_JWT_SECRET   = # не трогать
OFFLINE_MODE     = false

[mailer]
# выставить нужные значения для работы email
#ENABLED     = true
MAILER_TYPE = smtp
#HOST        = smtp.yandex.ru:465
#USER        = 
#PASSWD      = 
#FROM        = 

# остальное по желанию
[service]
REGISTER_EMAIL_CONFIRM            = true
ENABLE_NOTIFY_MAIL                = true
DISABLE_REGISTRATION              = false
ALLOW_ONLY_EXTERNAL_REGISTRATION  = false
ENABLE_CAPTCHA                    = true
REQUIRE_SIGNIN_VIEW               = false
DEFAULT_KEEP_EMAIL_PRIVATE        = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = true
DEFAULT_ENABLE_TIMETRACKING       = true
NO_REPLY_ADDRESS                  = noreply.localhost

[picture]
DISABLE_GRAVATAR        = false
ENABLE_FEDERATED_AVATAR = true

[openid]
ENABLE_OPENID_SIGNIN = true
ENABLE_OPENID_SIGNUP = false

[session]
PROVIDER = file

[log]
MODE      = console
LEVEL     = info
ROOT_PATH = /var/lib/gitea/log
ROUTER    = console
```

### `/etc/nginx/sites-available/gitea-proxy.conf`
В server_name указать нужный домен; в proxy_pass поставить localhost с портом HTTP_PORT из app.ini
```
server {
    listen 80;
    listen [::]:80;
    
    server_name my.domain.com

    access_log /var/log/nginx/gitea-access.log;
    error_log /var/log/nginx/gitea-error.log;
    client_max_body_size 100M;
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```
