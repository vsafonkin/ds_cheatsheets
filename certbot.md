## **Certbot Cheatsheet**

---

### **Основные команды**

| Команда                                      | Описание                                                                 |
|----------------------------------------------|-------------------------------------------------------------------------|
| `certbot --version`                          | Проверить версию Certbot.                                               |
| `certbot certificates`                       | Показать список выданных сертификатов.                                  |
| `certbot delete --cert-name example.com`     | Удалить сертификат для домена.                                          |
| `certbot renew --dry-run`                    | Тестовый запуск обновления сертификатов (без реального обновления).     |
| `certbot renew`                              | Обновить все сертификаты, срок действия которых истекает.               |
| `certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem`| Отозвать сертификат.                  |

---

### **Получение сертификатов**

#### **Для Nginx**
```bash
sudo certbot --nginx -d example.com -d www.example.com
```
- Автоматически настраивает Nginx для использования HTTPS.

#### **Для Apache**
```bash
sudo certbot --apache -d example.com -d www.example.com
```
- Автоматически настраивает Apache для использования HTTPS.

#### **Вручную (без автоматической настройки веб-сервера)**
```bash
sudo certbot certonly --standalone -d example.com -d www.example.com
```
- Получает сертификат без изменения конфигурации веб-сервера.

#### **С использованием DNS-провайдера (для wildcard-сертификатов)**
```bash
sudo certbot certonly --dns-<провайдер> -d example.com -d *.example.com
```
- Пример для Cloudflare:
  ```bash
  sudo certbot certonly --dns-cloudflare -d example.com -d *.example.com
  ```

---

### **Автоматическое обновление сертификатов**

Certbot автоматически добавляет задачу в cron для обновления сертификатов. Вы можете вручную проверить обновление:
```bash
sudo certbot renew --dry-run
```

---

### **Настройка веб-сервера**

#### **Nginx**
Certbot автоматически добавляет SSL-конфигурацию в файлы Nginx. Пример конфигурации:
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

#### **Apache**
Certbot автоматически добавляет SSL-конфигурацию в файлы Apache. Пример конфигурации:
```apache
<VirtualHost *:443>
    ServerName example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    DocumentRoot /var/www/html
</VirtualHost>
```

---

### **Полезные команды**

| Команда                                      | Описание                                                                 |
|----------------------------------------------|-------------------------------------------------------------------------|
| `sudo certbot renew --force-renewal`         | Принудительно обновить все сертификаты.                                 |
| `sudo certbot renew --post-hook "systemctl restart nginx"`| Перезапустить Nginx после обновления сертификатов.               |
| `sudo certbot renew --pre-hook "systemctl stop nginx"`| Остановить Nginx перед обновлением сертификатов.                  |
| `sudo certbot plugins`                       | Показать список доступных плагинов.                                     |

---

### **Управление сертификатами**

| Команда                                      | Описание                                                                 |
|----------------------------------------------|-------------------------------------------------------------------------|
| `sudo certbot delete --cert-name example.com`| Удалить сертификат для домена.                                          |
| `sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem`| Отозвать сертификат.                  |
| `sudo certbot update_symlinks`               | Обновить символические ссылки на сертификаты.                           |

---

### **Примеры использования**

#### **Получение сертификата для нескольких доменов**
```bash
sudo certbot --nginx -d example.com -d www.example.com -d api.example.com
```

#### **Получение wildcard-сертификата**
```bash
sudo certbot certonly --dns-cloudflare -d example.com -d *.example.com
```

#### **Обновление сертификатов с перезапуском Nginx**
```bash
sudo certbot renew --post-hook "systemctl restart nginx"
```

---

### **Заключение**
Этот cheatsheet охватывает основные команды и сценарии использования Certbot для получения, обновления и управления SSL-сертификатами. Используйте его как справочник для быстрой настройки HTTPS на вашем сервере.
