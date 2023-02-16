Реверс-прокси
```ini
server {
    listen  80;
    server_name 8.8.8.8 default_server;
    proxy_http_version 1.1;

    location / {
      proxy_pass https://package.gitlab.com;
      resolver 8.8.8.8 valid=30s ipv4=on ipv6=off;
      proxy_ssl_verify       off;
      proxy_ssl_server_name  on;
    }
  }
```
