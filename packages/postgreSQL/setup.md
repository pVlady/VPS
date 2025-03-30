# Установка PostgreSQL

## Ubuntui 24.04 LTS (noble)

```bash
# подключаем репозиторий
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/noble-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

wget --quiet -O - https://postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# проверить локаль
locale
# для русского языка
export LC_CTYPE=ru_RU.UTF8
export LC_COLLATE=ru_RU.UTF8

locale -a | grep ru_RU      ; проверить, что в ОС установлена локаль ru_RU
sudo locale-gen ru_RU.utf8  ; сгенерировать, если отсутствует

sudo apt install postgresql-17              ; установка PostgreSQL
sudo -u postgres psql -c 'select version()' ; проверка подключения
```
Служба postgresql.service будет автоматически запущена.
Журнал логов находится в */var/lib/postgres/17/main*.

Посмотреть справку в psql по командам SQL можно командой `\help CREATE TABLE`

### Конфигурация сервера базы данных
#### `postgresql.conf`
После изменения значения параметра или добавления новой строки нужно дать серверу команду перечитать настройки:
```psql
postgres=# SELECT pg_reload_conf();
SHOW <param>   ; показать значение параметра (если не изменилось, значит допущена ошибка, см.логи)
```
>Если настройка указана в файле несколько раз, то будет использоваться самое последнее присвоение значения параметру.

Значение парметра также можно изменить SQL-запросом:
```sql
postgres=# ALTER SYSTEM SET work_mem='128MB';
```





