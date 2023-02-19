# Получение SSL-сертификатов
```bash
sudo letsencrypt certonly -a webroot --webroot-path=/var/www/html/git.example.com -d git.example.com -d www.git.example.com
```
