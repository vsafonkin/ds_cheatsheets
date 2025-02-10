Для защиты эндпоинта `/metrics` и ограничения доступа только для определенного экземпляра Prometheus можно использовать несколько подходов. Самые распространенные из них:

1. **Базовая аутентификация (HTTP Basic Auth)**.
2. **IP-фильтрация**.
3. **Использование токенов (например, через заголовки)**.

Рассмотрим каждый из этих методов.

---

## **1. Базовая аутентификация (HTTP Basic Auth)**

Этот метод требует, чтобы Prometheus передавал логин и пароль при запросе метрик.

### Шаги для настройки:

#### 1. Создайте файл с паролями
Используйте утилиту `htpasswd` для создания файла с логинами и паролями:
```bash
sudo sh -c "echo -n 'prometheus_user:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```
Введите пароль для пользователя `prometheus_user`.

#### 2. Настройте Nginx
Добавьте в конфигурацию Nginx блок для `/metrics` с базовой аутентификацией:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location /metrics {
        alias /var/www/metrics.txt;
        default_type text/plain;

        auth_basic "Prometheus Metrics";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

#### 3. Настройте Prometheus
В конфигурации Prometheus (`prometheus.yml`) укажите логин и пароль для доступа к эндпоинту:
```yaml
scrape_configs:
  - job_name: 'nginx_metrics'
    basic_auth:
      username: prometheus_user
      password: your_password
    static_configs:
      - targets: ['your_domain_or_ip:80']
```

#### 4. Перезапустите Nginx и Prometheus
```bash
sudo systemctl reload nginx
sudo systemctl restart prometheus
```

---

## **2. IP-фильтрация**

Этот метод ограничивает доступ к эндпоинту `/metrics` только для определенного IP-адреса (например, IP-адреса вашего сервера Prometheus).

### Шаги для настройки:

#### 1. Настройте Nginx
Добавьте в конфигурацию Nginx блок для `/metrics` с ограничением по IP:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location /metrics {
        alias /var/www/metrics.txt;
        default_type text/plain;

        allow 192.168.1.100;  # IP-адрес вашего Prometheus
        deny all;
    }
}
```

#### 2. Перезапустите Nginx
```bash
sudo systemctl reload nginx
```

---

## **3. Использование токенов (заголовки)**

Этот метод требует, чтобы Prometheus передавал специальный токен в заголовке запроса.

### Шаги для настройки:

#### 1. Настройте Nginx
Добавьте проверку заголовка в конфигурацию Nginx:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location /metrics {
        alias /var/www/metrics.txt;
        default_type text/plain;

        if ($http_x_metrics_token != "your_secret_token") {
            return 403;
        }
    }
}
```

#### 2. Настройте Prometheus
В конфигурации Prometheus укажите заголовок с токеном:
```yaml
scrape_configs:
  - job_name: 'nginx_metrics'
    static_configs:
      - targets: ['your_domain_or_ip:80']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __metrics_path__
        replacement: /metrics
    headers:
      X-Metrics-Token: "your_secret_token"
```

#### 3. Перезапустите Nginx и Prometheus
```bash
sudo systemctl reload nginx
sudo systemctl restart prometheus
```

---

## **4. Комбинированный подход**

Для большей безопасности можно комбинировать несколько методов. Например:
- Использовать базовую аутентификацию и IP-фильтрацию.
- Использовать токены и IP-фильтрацию.

Пример комбинированной настройки:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location /metrics {
        alias /var/www/metrics.txt;
        default_type text/plain;

        allow 192.168.1.100;  # IP-адрес вашего Prometheus
        deny all;

        auth_basic "Prometheus Metrics";
        auth_basic_user_file /etc/nginx/.htpasswd;

        if ($http_x_metrics_token != "your_secret_token") {
            return 403;
        }
    }
}
```

---

## **Заключение**
- **Базовая аутентификация** — простой и надежный способ, но требует передачи логина и пароля.
- **IP-фильтрация** — подходит, если IP-адрес Prometheus статичен.
- **Токены** — более гибкий способ, но требует настройки заголовков.
