# Nginx Web Server

Файл конфигурации *nginx.conf* с поддержкой fpm-php:
```txt
events {}
http {
  server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;

    # проксирование shiny-приложения
    location /1st {
      proxy_pass http://host.docker.internal:3838/1st;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }

    # поддержка php
    location ~ \.php$ {
      try_files $uri =404;
      fastcgi_pass php:9000;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
    }
  }
}
```

Файл docker-compose.yml для запуска nginx в контейнере:
```yaml
services:
  nginx:
    image: nginx:latest
    extra_hosts:
      - "host.docker.internal:host-gateway"
    ports:
      - "80:80"
    volumes:
      - /usr/local/etc/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /var/www/html:/usr/share/nginx/html
    depends_on:
      - php
  php:
    image: php:8.3-fpm
    volumes:
      - /var/www/html:/usr/share/nginx/html
```

Для автоматического запуска и остановки веб-сервера нужно создать службу
docker-nginx.service в каталоге /etc/systemd/system и запустить ее:
```ini
[Unit]
Description=Nginx Web Server
After=network.target

[Service]
ExecStart=docker compose -f /usr/local/bin/nginx/docker-compose.yml up
ExecStop=docker compose -f /usr/local/bin/nginx/docker-compose.yml down

[Install]
WantedBy=default.target
```
```bash
systemctl enable docker-nginx
systemctl start docker-nginx   ; проверка запуска службы
systemctl stop docker-nginx    ; проверка останоки службы
```
