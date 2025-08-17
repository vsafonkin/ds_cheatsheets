## 🔹 1. **Базовые команды**

| Команда | Описание |
|--------|--------|
| `tcpdump -i eth0` | Сниффинг трафика на интерфейсе `eth0` |
| `tcpdump -i any` | Сниффинг на всех интерфейсах |
| `tcpdump -i lo` | Сниффинг на loopback (важно: может не работать на `lo` без `-p`) |
| `tcpdump -c 10` | Остановить после 10 пакетов |
| `tcpdump -n` | Не разрешать IP и порты в имена (быстрее, без DNS) |
| `tcpdump -nn` | Без разрешения IP и портов (порт как число, не как `http`) |
| `tcpdump -v` | Подробный вывод (можно `-vv`, `-vvv`) |
| `tcpdump -q` | Краткий (quiet) вывод |
| `tcpdump -X` | Показать пакет в hex и ASCII |
| `tcpdump -XX` | То же, но с заголовками Ethernet |
| `tcpdump -s 0` | Захватывать полный размер пакета (snaplen = 65535) |

> 💡 `-s 0` или `-s 1500` — важно, если нужно видеть **весь payload**, а не обрезанные куски.

---

## 🔹 2. **Фильтрация по сетевым параметрам**

| Фильтр | Описание |
|--------|--------|
| `tcpdump host 192.168.1.10` | Только трафик с/на IP |
| `tcpdump src 192.168.1.10` | Только исходящий от IP |
| `tcpdump dst 192.168.1.10` | Только входящий к IP |
| `tcpdump net 192.168.1.0/24` | Трафик в сети |
| `tcpdump net 192.168.1.0 mask 255.255.255.0` | С маской (старый синтаксис) |

---

## 🔹 3. **Фильтрация по протоколам**

| Фильтр | Описание |
|--------|--------|
| `tcpdump tcp` | Только TCP |
| `tcpdump udp` | Только UDP |
| `tcpdump icmp` | Только ICMP (ping) |
| `tcpdump arp` | Только ARP-запросы |
| `tcpdump ip6` | Только IPv6 |
| `tcpdump port 80` | Только трафик на/с порта 80 |
| `tcpdump src port 53` | Исходящий с порта 53 (DNS) |
| `tcpdump dst port 443` | Направленный на порт 443 |
| `tcpdump portrange 8000-9000` | В диапазоне портов |

---

## 🔹 4. **Комбинированные фильтры (логические операторы)**

| Фильтр | Описание |
|--------|--------|
| `tcpdump tcp and port 80` | TCP и порт 80 |
| `tcpdump host 192.168.1.10 and not ssh` | От/к IP, кроме SSH |
| `tcpdump src 192.168.1.10 and dst port 443` | От IP к порту 443 |
| `tcpdump udp or icmp` | UDP или ICMP |
| `tcpdump '(tcp port 80) and (host 10.0.0.5)'` | Скобки группируют условия |
| `tcpdump -Q in` | Только входящий трафик (если поддерживается) |
| `tcpdump -Q out` | Только исходящий |

> ⚠️ При использовании `and`, `or`, `not` — лучше брать фильтр в **кавычки**, чтобы обойти shell.

---

## 🔹 5. **Фильтрация по типу TCP-флагов**

| Фильтр | Описание |
|--------|--------|
| `tcpdump 'tcp[tcpflags] == tcp-syn'` | Только SYN-пакеты |
| `tcpdump 'tcp[tcpflags] & tcp-syn != 0'` | Любой пакет с установленным SYN |
| `tcpdump 'tcp[tcpflags] == tcp-rst'` | Только RST |
| `tcpdump 'tcp[tcpflags] & tcp-ack == 0'` | Пакеты без ACK (например, SYN) |
| `tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin|tcp-rst) != 0'` | SYN, FIN или RST (управление соединением) |

---

## 🔹 6. **Сохранение и чтение дампов**

| Команда | Описание |
|--------|--------|
| `tcpdump -w capture.pcap` | Записать в файл (бинарный pcap) |
| `tcpdump -r capture.pcap` | Прочитать из файла |
| `tcpdump -r capture.pcap port 80` | Читать и фильтровать дамп |
| `tcpdump -w - \| gzip > capture.pcap.gz` | Запись с компрессией |
| `gunzip -c capture.pcap.gz \| tcpdump -r -` | Чтение сжатого дампа |

---

## 🔹 7. **Полезные комбинации (реальные примеры)**

| Задача | Команда |
|-------|--------|
| **HTTP-трафик** | `tcpdump -i eth0 -nn -s 0 -v port 80` |
| **HTTPS (TLS)** | `tcpdump -i eth0 -nn -s 0 port 443` |
| **DNS-запросы** | `tcpdump -i eth0 -nn port 53` |
| **Пинги (ICMP)** | `tcpdump -i eth0 icmp` |
| **TCP handshake (SYN)** | `tcpdump 'tcp[tcpflags] & tcp-syn != 0'` |
| **Найти сканер портов** | `tcpdump 'tcp[tcpflags] == tcp-syn' and 'tcp[13] == 2'` (SYN без ACK) |
| **Трафик к определённому серверу** | `tcpdump host api.example.com and port 443` |
| **Весь трафик, кроме SSH** | `tcpdump not port 22` |
| **Только broadcast/ARP** | `tcpdump ether broadcast or arp` |

---

## 🔹 8. **Продвинутые опции**

| Опция | Описание |
|------|--------|
| `tcpdump -e` | Показывать Ethernet-заголовки (MAC-адреса) |
| `tcpdump -E secret` | Расшифровка IPsec (устарело, лучше использовать `tcpflow` или Wireshark) |
| `tcpdump -A` | Показать данные в ASCII (для текста, например HTTP) |
| `tcpdump -x` | Данные в hex (без заголовков) |
| `tcpdump -D` | Показать все доступные интерфейсы |
| `tcpdump -l` | Line-buffered вывод (полезно при пайпе в `grep`) |
| `tcpdump -t` | Убрать временные метки |
| `tcpdump -tttt` | Полная дата и время (человекочитаемо) |

Пример:
```bash
tcpdump -i eth0 -l port 80 | grep "GET"
```
— позволяет фильтровать HTTP-запросы в реальном времени.

---

## 🔹 9. **Ограничение размера и времени**

| Опция | Описание |
|------|--------|
| `tcpdump -c 100 -w dump.pcap` | 100 пакетов в файл |
| `timeout 30 tcpdump -w temp.pcap` | Запись 30 секунд |
| `tcpdump -G 60 -w file-%S.pcap` | Ротация файла каждые 60 секунд |
| `tcpdump -C 1 -w bigfile.pcap` | Новый файл при достижении ~1MB |

---

## 🔹 10. **Безопасность и советы**

- Запускайте `tcpdump` с `sudo` или дайте CAP_NET_RAW:
  ```bash
  sudo setcap cap_net_raw+ep /usr/sbin/tcpdump
  ```
- Не используйте `tcpdump` на продакшене без фильтрации — может "задушить" систему.
- Для анализа pcap-файлов лучше использовать **Wireshark**, **tshark**, **tcpflow**.
- Избегайте `tcpdump -i any` при анализе — может дублировать пакеты.

---

## 📌 Пример: диагностика подключения

```bash
# Смотрим, идёт ли SYN к веб-серверу
tcpdump -i eth0 -nn 'tcp port 80 and (tcp-syn or tcp-ack or tcp-rst)'
```

Если видите `SYN`, но нет `SYN-ACK` — проблема на стороне сервера или в сети.

---

## 🧰 Полезные альтернативы
- `tshark` — CLI-версия Wireshark, мощнее, но сложнее.
- `wireshark` — GUI, для глубокого анализа.
- `ngrep` — как `grep`, но по сетевому трафику.
- `tcpflow` — восстанавливает потоки данных из pcap.
