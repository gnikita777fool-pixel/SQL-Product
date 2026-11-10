### 🔹 Шаг 1: Подготовка окружения и запуск базы данных

1. Откройте **Git Bash** (или терминал в VS Code).
2. Очистите систему от старых тестовых контейнеров и томов (чтобы не было конфликтов):
   ```bash
   docker container prune -f
   docker volume prune -f
   ```
3. Создайте рабочую папку и перейдите в неё:
   ```bash
   mkdir products-mysql
   cd products-mysql
   ```
4. Создайте файл настроек `my.cnf` для корректной работы с кириллицей. В VS Code создайте новый файл `my.cnf` в этой папке и вставьте туда этот текст:
   ```ini
   [mysqld]
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   [client]
   default-character-set = utf8mb4
   [mysql]
   default-character-set = utf8mb4
   ```
5. Запустите контейнер с MySQL (пароль root будет `123`):
   ```bash
   docker run --name products-mysql \
     -e MYSQL_ROOT_PASSWORD=123 \
     -e LANG=C.UTF-8 \
     -v mysql_data:/var/lib/mysql \
     -v $(pwd)/my.cnf:/etc/mysql/conf.d/my.cnf \
     -d mysql:latest
   ```
6. Проверьте, что контейнер работает:
   ```bash
   docker ps
   ```
7. Войдите внутрь контейнера в консоль MySQL:
   ```bash
   docker exec -it products-mysql mysql --default-character-set=utf8mb4 -uroot -p
   ```
   *(Когда попросит пароль, введите `123`)*
8. Создайте базу данных и выберите её:
   ```sql
   CREATE DATABASE shop CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   USE shop;
   ```

---

### 🔹 Шаг 2: Создание таблиц

Выполните эти команды в консоли MySQL:
```sql
CREATE TABLE products1 (
    name  VARCHAR(100) NOT NULL,
    price INT NOT NULL
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE products2 (
    name  VARCHAR(100) NOT NULL,
    price INT NOT NULL
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

SHOW TABLES; -- Проверка, что таблицы создались
```
<img width="589" height="424" alt="image" src="https://github.com/user-attachments/assets/31aa4825-3cdc-4d27-ba3f-fb15cc39a331" />

---

### 🔹 Шаг 3: Вставка данных

Заполните таблицы тестовыми данными:
```sql
INSERT INTO products1 (name, price) VALUES
  ('Дыни', 800), ('Клубника', 150), ('Яблоки', 120), ('Лимоны', 200);

INSERT INTO products2 (name, price) VALUES
  ('Дыни', 800), ('Клубника', 150), ('Каштаны', 200), ('Хурма', 350);

-- Проверка
SELECT * FROM products1;
SELECT * FROM products2;
```

<img width="664" height="630" alt="image" src="https://github.com/user-attachments/assets/88743208-fa52-4fd6-9cf0-662d37b3f2b6" />

---

### 🔹 Шаг 4: Основные SQL-операции (Делайте скриншоты результатов!)

Выполняйте запросы по очереди и фиксируйте результат:

1. **Выборка (фильтрация)**: Товары из `products1` дороже 150.
   ```sql
   SELECT * FROM products1 WHERE price > 150;
   ```
   <img width="516" height="173" alt="image" src="https://github.com/user-attachments/assets/fbe2f442-3b6c-4764-8efd-87152dcf6e0c" />

2. **Проекция**: Только названия товаров.
   ```sql
   SELECT name FROM products1;
   ```
   <img width="434" height="166" alt="image" src="https://github.com/user-attachments/assets/4566df30-b1d5-4bf5-abbc-95d1222c43d3" />

3. **Объединение (UNION)**: Уникальный список всех товаров из обеих таблиц.
   ```sql
   SELECT name FROM products1 UNION SELECT name FROM products2;
   ```
   <img width="638" height="217" alt="image" src="https://github.com/user-attachments/assets/c83ad1f6-f863-481b-8a57-f704a84f1129" />

4. **Разность (EXCEPT)**: Есть в первой, но нет во второй (работает в MySQL 8.0.31+).
   ```sql
   SELECT name, price FROM products1 EXCEPT SELECT name, price FROM products2;
   ```
   <img width="723" height="170" alt="image" src="https://github.com/user-attachments/assets/9fab9418-960b-4837-b6ad-d3efb73fb320" />

5. **Пересечение (INTERSECT)**: Одинаковые товары с одинаковой ценой в обеих таблицах.
   ```sql
   SELECT name, price FROM products1 INTERSECT SELECT name, price FROM products2;
   ```
   <img width="733" height="170" alt="image" src="https://github.com/user-attachments/assets/67701898-029b-4c2e-93c0-cf8860a4ae5d" />

6. **Декартово произведение**: Все возможные пары товаров.
   ```sql
   SELECT t1.name AS name1, t1.price AS price1, t2.name AS name2, t2.price AS price2
   FROM products1 t1 CROSS JOIN products2 t2;
   ```
   <img width="791" height="412" alt="image" src="https://github.com/user-attachments/assets/74d88977-23b2-43e7-bb59-fc701f83f775" />

7. **Соединение (JOIN)**: Сравнение цен для одинаковых товаров.
   ```sql
   SELECT t1.name, t1.price AS price1, t2.price AS price2
   FROM products1 t1 JOIN products2 t2 ON t1.name = t2.name;
   ```
   <img width="569" height="181" alt="image" src="https://github.com/user-attachments/assets/05036a58-68b5-4617-8321-42ef92352126" />

   **LEFT JOIN** (все товары из первой таблицы, даже если нет совпадений во второй):
   ```sql
   SELECT t1.name, t1.price AS price1, t2.price AS price2
   FROM products1 t1 LEFT JOIN products2 t2 ON t1.name = t2.name;
   ```
   <img width="760" height="246" alt="image" src="https://github.com/user-attachments/assets/cda7f832-17d6-411f-bdcc-2359621de5a9" />

---

### 🔹 Шаг 5: Создание дампа (Резервная копия)

1. Выйдите из консоли MySQL:
   ```sql
   \q
   ```
2. Находясь в обычном терминале (Git Bash), создайте дамп базы данных в файл:
   ```bash
   docker exec -e MYSQL_PWD=123 products-mysql mysqldump -uroot shop > dump.sql
   ```
3. Убедитесь, что файл появился и не пустой:
   ```bash
   cat dump.sql
   ```
   *(или просто откройте файл `dump.sql` в VS Code)*.

---

### 🔹 Шаг 6: Имитация потери данных (Уборка)

1. Снова войдите в MySQL:
   ```bash
   docker exec -it products-mysql mysql --default-character-set=utf8mb4 -uroot -p
   ```
   *(пароль: `123`)*
2. Удалите базу данных:
   ```sql
   USE shop;
   DROP DATABASE shop;
   SHOW DATABASES; -- Убедитесь, что shop исчезла из списка
   \q
   ```

---

### 🔹 Шаг 7: Восстановление БД из дампа

1. Снова войдите в MySQL и создайте **пустую** базу данных:
   ```bash
   docker exec -it products-mysql mysql --default-character-set=utf8mb4 -uroot -p
   ```
   ```sql
   CREATE DATABASE shop CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   \q
   ```
2. Восстановите данные из файла `dump.sql` (выполняется в **Git Bash**, а не в MySQL!):
   ```bash
   docker exec -i products-mysql mysql -uroot -p123 --force shop < dump.sql
   ```
3. Проверьте успешность восстановления:
   ```bash
   docker exec -it products-mysql mysql --default-character-set=utf8mb4 -uroot -p
   ```
   ```sql
   USE shop;
   SHOW TABLES;
   SELECT * FROM products1;
   SELECT * FROM products2;
   \q
   ```

---
