Скачать бинарный файл *gitea*, сделать его исполняемыи и поместить в каталог */usr/local/bin*
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
Cкачать файл gitea.service и отредактировать его (раскомментировать нужную БД, указать пользователя и пароль), после чего запустить сервис
```
sudo wget https://raw.githubusercontent.com/go-gitea/gitea/master/contrib/systemd/gitea.service -P /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now gitea
```
По умолчанию gitea прослушивает соединения на порту 3000 всех сетевых интерфейсов, поэтому необъодимо в брандмауэре открыть 3000/tcp:
```
sudo ufw allow 3000/tcp
```
Далее перейти по адресу `https://ip_addr:3000` и завершить установку.\
Первый зарегистрированный пользователь автоматически добавляется в группу администраторов.
Пример настройки базы данных (sqlite3):
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

После завершения установки необходимо изменить разрешения для */etc/gitea*
```
chmod 750 /etc/gitea
chmod 640 /etc/gitea/app.ini
```
