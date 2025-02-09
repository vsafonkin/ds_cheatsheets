Настройка обновления SSL-сертификатов для Nginx, запущенного в контейнере, с использованием Certbot требует некоторых дополнительных шагов, так как Certbot и Nginx находятся в разных контейнерах. Вот пошаговое руководство:

---

## **Шаги для настройки обновления сертификатов**

### 1. **Создание Docker-сети**
   Создайте пользовательскую Docker-сеть, чтобы контейнеры могли взаимодействовать друг с другом:
   ```bash
   docker network create nginx-net
   ```

---

### 2. **Запуск контейнера Nginx**
   Запустите контейнер Nginx, подключив его к созданной сети и монтируя директорию с сертификатами:
   ```bash
   docker run -d \
     --name nginx \
     --network nginx-net \
     -p 80:80 \
     -p 443:443 \
     -v /etc/letsencrypt:/etc/letsencrypt \
     -v /var/www/html:/usr/share/nginx/html \
     nginx:latest
   ```

   - `/etc/letsencrypt` — монтируется для доступа к сертификатам.
   - `/var/www/html` — монтируется для размещения файлов проверки Let's Encrypt.

---

### 3. **Настройка Nginx для Certbot**
   Настройте Nginx для работы с Certbot. Создайте конфигурационный файл для домена:
   ```nginx
   server {
       listen 80;
       server_name example.com;

       location /.well-known/acme-challenge/ {
           root /var/www/html;
       }

       location / {
           return 301 https://$host$request_uri;
       }
   }
   ```

   Скопируйте конфигурацию в контейнер:
   ```bash
   docker cp nginx.conf nginx:/etc/nginx/conf.d/default.conf
   ```

   Перезапустите контейнер Nginx:
   ```bash
   docker restart nginx
   ```

---

### 4. **Получение сертификата с помощью Certbot**
   Запустите контейнер Certbot для получения сертификата:
   ```bash
   docker run -it --rm \
     --network nginx-net \
     -v /etc/letsencrypt:/etc/letsencrypt \
     -v /var/www/html:/var/www/html \
     certbot/certbot certonly --webroot \
     -w /var/www/html -d example.com
   ```

   - `--webroot` — указывает, что Certbot будет использовать веб-сервер для проверки домена.
   - `-w /var/www/html` — путь к корневой директории веб-сервера.

---

### 5. **Настройка Nginx для HTTPS**
   Обновите конфигурацию Nginx для использования SSL:
   ```nginx
   server {
       listen 443 ssl;
       server_name example.com;

       ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

       location / {
           root /usr/share/nginx/html;
           index index.html;
       }
   }
   ```

   Скопируйте обновлённую конфигурацию в контейнер:
   ```bash
   docker cp nginx-ssl.conf nginx:/etc/nginx/conf.d/default.conf
   ```

   Перезапустите контейнер Nginx:
   ```bash
   docker restart nginx
   ```

---

### 6. **Автоматическое обновление сертификатов**
   Для автоматического обновления сертификатов создайте скрипт и добавьте его в cron.

#### **Скрипт для обновления**
   Создайте скрипт `renew-certs.sh`:
   ```bash
   #!/bin/bash
   docker run --rm \
     --network nginx-net \
     -v /etc/letsencrypt:/etc/letsencrypt \
     -v /var/www/html:/var/www/html \
     certbot/certbot renew --webroot -w /var/www/html

   docker restart nginx
   ```

   Сделайте скрипт исполняемым:
   ```bash
   chmod +x renew-certs.sh
   ```

#### **Добавление в cron**
   Откройте crontab:
   ```bash
   crontab -e
   ```

   Добавьте задачу для ежедневного обновления:
   ```bash
   0 0 * * * /path/to/renew-certs.sh
   ```

---

### 7. **Проверка обновления**
   Проверьте, что обновление работает:
   ```bash
   sudo certbot renew --dry-run
   ```

---

### **Итог**
- Certbot и Nginx работают в отдельных контейнерах, но используют общие директории для сертификатов и файлов проверки.
- Автоматическое обновление сертификатов настраивается через cron и скрипт.
- Этот подход позволяет легко масштабировать и управлять SSL-сертификатами в контейнеризованной среде.
