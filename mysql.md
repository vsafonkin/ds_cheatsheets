В MariaDB (и MySQL) список пользователей и их привилегии хранятся в системной базе данных `mysql`. Чтобы посмотреть список пользователей, у которых есть доступ к конкретной базе данных, можно использовать SQL-запросы к таблицам `mysql.user`, `mysql.db` и другим.

---

### **1. Посмотреть всех пользователей**

Чтобы получить список всех пользователей, выполните следующий запрос:
```sql
SELECT User, Host FROM mysql.user;
```

Этот запрос покажет всех пользователей и хосты, с которых они могут подключаться.

---

### **2. Посмотреть пользователей с доступом к конкретной базе данных**

Чтобы узнать, какие пользователи имеют доступ к конкретной базе данных (например, `my_database`), выполните запрос к таблице `mysql.db`:
```sql
SELECT User, Host, Db FROM mysql.db WHERE Db = 'my_database';
```

Этот запрос покажет:
- `User` — имя пользователя.
- `Host` — хост, с которого пользователь может подключаться.
- `Db` — имя базы данных, к которой есть доступ.

---

### **3. Посмотреть привилегии пользователя для конкретной базы данных**

Чтобы узнать, какие привилегии есть у конкретного пользователя для базы данных, используйте команду `SHOW GRANTS`:
```sql
SHOW GRANTS FOR 'username'@'host';
```

Пример:
```sql
SHOW GRANTS FOR 'my_user'@'localhost';
```

Этот запрос покажет все привилегии, которые есть у пользователя `my_user` для всех баз данных, включая глобальные привилегии.

---

### **4. Посмотреть привилегии для конкретной базы данных**

Если вам нужно узнать, какие пользователи имеют доступ к конкретной базе данных и какие у них привилегии, можно объединить данные из таблиц `mysql.user` и `mysql.db`:
```sql
SELECT 
    u.User, 
    u.Host, 
    db.Db, 
    db.Select_priv, 
    db.Insert_priv, 
    db.Update_priv, 
    db.Delete_priv 
FROM 
    mysql.user u
JOIN 
    mysql.db db 
ON 
    u.User = db.User AND u.Host = db.Host
WHERE 
    db.Db = 'my_database';
```

Этот запрос покажет:
- `User` — имя пользователя.
- `Host` — хост.
- `Db` — имя базы данных.
- `Select_priv`, `Insert_priv`, `Update_priv`, `Delete_priv` — привилегии на выполнение соответствующих операций.

---

### **5. Посмотреть глобальные привилегии**

Некоторые пользователи могут иметь глобальные привилегии (например, `ALL PRIVILEGES`), которые распространяются на все базы данных. Чтобы проверить это, выполните:
```sql
SELECT User, Host, Super_priv, Create_priv, Drop_priv FROM mysql.user;
```

- `Super_priv` — наличие привилегий суперпользователя.
- `Create_priv` — возможность создавать базы данных.
- `Drop_priv` — возможность удалять базы данных.

---

### **6. Использование команды `SHOW DATABASES`**

Если вы хотите узнать, к каким базам данных имеет доступ текущий пользователь, выполните:
```sql
SHOW DATABASES;
```

Это покажет список баз данных, к которым у пользователя есть доступ.

---

### **Итог:**
- Используйте таблицы `mysql.user` и `mysql.db`, чтобы узнать список пользователей и их привилегии.
- Команда `SHOW GRANTS` позволяет быстро посмотреть привилегии конкретного пользователя.
- Для детального анализа привилегий можно использовать SQL-запросы к системным таблицам.

Теперь вы сможете легко проверить, кто имеет доступ к вашей базе данных и какие у них права!
