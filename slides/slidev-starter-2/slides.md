---
theme: light-icons
title: SQL основы в PostgreSQL
background: https://i.pinimg.com/736x/ce/97/ee/ce97eee5a58b1a2544838846bfab69ea.jpg
info: |
  ## SQL основы: DDL и DML
  Презентация по основам SQL в PostgreSQL
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
author: PostgreSQL
routerMode: hash
layout: intro
image: https://i.pinimg.com/736x/2f/7a/30/2f7a30688d08269ced43f3025da00dcc.jpg
---

  <div class="mb-4 absolute bottom-16 left-12">
    <span class="text-6xl font-extrabold text-primary-lighter text-opacity-80" >
        SQL основы в PostgreSQL
    </span>
    <div class="text-2xl mt-4 text-white text-left text-opacity-80" style="font-weight:600;" >
        DDL и DML операции, создание таблиц и связей
    </div> 
  </div>
  
---

# Что такое SQL?

Structured Query Language

**SQL** — язык структурированных запросов для работы с реляционными базами данных.

<v-click>

В этом уроке мы изучим:

- ✅ Операции определения данных (DDL)
- ✅ Операции манипулирования данными (DML)
- ✅ Создание таблиц и связей

</v-click>

---

## Категории SQL команд

Четыре основных типа

| Категория | Описание | Основные команды |
|-----------|----------|------------------|
| **DDL** | Определение структуры данных | CREATE, ALTER, DROP, TRUNCATE |
| **DML** | Манипулирование данными | SELECT, INSERT, UPDATE, DELETE |
| **DCL** | Управление доступом | GRANT, REVOKE |
| **TCL** | Управление транзакциями | BEGIN, COMMIT, ROLLBACK, SAVEPOINT |

<v-click>

В этом уроке фокус на **DDL** и **DML** — это 90% повседневной работы с БД.

</v-click>

---

# CREATE DATABASE

Создание базы данных

```sql {all|2|5-12|15|all}
-- Простое создание
CREATE DATABASE mydb;

-- С полными параметрами
CREATE DATABASE sales_db
    WITH 
    OWNER = sales_admin
    ENCODING = 'UTF8'
    LC_COLLATE = 'ru_RU.UTF-8'
    LC_CTYPE = 'ru_RU.UTF-8'
    CONNECTION LIMIT = 100
    IS_TEMPLATE = False;

-- С комментарием
COMMENT ON DATABASE sales_db IS 'База данных отдела продаж';
```

<v-click>

💡 **Всегда используйте UTF8** кодировку для поддержки международных символов!

</v-click>

---

# CREATE TABLE: базовый синтаксис

Создание таблицы

```sql{all}
CREATE TABLE table_name (
    column1 datatype [constraint],
    column2 datatype [constraint],
    ...
    [table_constraint]
);
```

<v-click>

### Простой пример

```sql {all|2|3-5|6|7|8|all}
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

</v-click>

---

# Типы данных: числовые

Целые числа и десятичные

```sql {all|2-6|6-9|11-14|15-17|all}
CREATE TABLE numeric_examples (
    -- Целые числа
    tiny_int SMALLINT,              -- -32768 до 32767
    normal_int INTEGER,             -- -2147483648 до 2147483647
    big_int BIGINT,                 -- очень большие числа
    
    -- Автоинкремент
    id SERIAL,                      -- INTEGER с auto-increment
    big_id BIGSERIAL,               -- BIGINT с auto-increment
    
    -- Десятичные
    price NUMERIC(10, 2),           -- точные вычисления (10 цифр, 2 после запятой)
    percentage NUMERIC(5, 2),       -- 999.99
    
    -- Плавающая точка
    scientific REAL,                -- 6 знаков точности
    precise DOUBLE PRECISION        -- 15 знаков точности
);
```

---

# Типы данных: строковые

Текст и бинарные данные

```sql {all|2-4|7|all}
CREATE TABLE string_examples (
    fixed_length CHAR(10),          -- точно 10 символов
    var_length VARCHAR(100),        -- до 100 символов
    unlimited TEXT,                 -- без ограничений
    
    -- Бинарные данные
    binary_data BYTEA               -- бинарные данные
);
```

<v-click>

### Когда что использовать?

- **CHAR(n)** — когда длина всегда фиксирована (коды, индексы)
- **VARCHAR(n)** — когда есть разумное ограничение (email, имена)
- **TEXT** — когда длина непредсказуема (описания, контент)

</v-click>

---

# Типы данных: временные

Даты и время

```sql {all|2-4|6-7|9-10|all}
CREATE TABLE datetime_examples (
    date_only DATE,                             -- только дата
    time_only TIME,                             -- только время
    time_with_tz TIME WITH TIME ZONE,           -- время с часовым поясом
    
    timestamp_val TIMESTAMP,                    -- дата и время
    timestamp_tz TIMESTAMPTZ,                   -- с часовым поясом (рекомендуется!)
    
    -- Интервалы
    duration INTERVAL                           -- временной интервал
);
```

<v-click>

⚠️ **Важно**: Всегда используйте **TIMESTAMPTZ** вместо TIMESTAMP для хранения моментов времени, чтобы избежать проблем с часовым поясом!

</v-click>

---

# Специальные типы данных

JSON, массивы, UUID и другие

```sql {all|2-3|5-7|9-11|13-17|18-20|all}
CREATE TABLE special_types (
    -- UUID
    unique_id UUID DEFAULT gen_random_uuid(),
    
    -- JSON
    metadata JSON,                  -- JSON (текст)
    settings JSONB,                 -- JSON Binary (индексируемый)
    
    -- Массивы
    tags TEXT[],                    -- массив строк
    numbers INTEGER[],              -- массив чисел
    
    -- Диапазоны
    age_range INT4RANGE,            -- диапазон целых чисел
    price_range NUMRANGE,           -- диапазон NUMERIC
    date_range DATERANGE,           -- диапазон дат
    
    -- Сетевые адреса
    ip_address INET,                -- IPv4 или IPv6
    mac_address MACADDR             -- MAC адрес
);
```

---

# Ограничения (Constraints)

Правила для обеспечения целостности данных

**Constraints** — это правила, которые ограничивают данные в таблице для обеспечения корректности.

```sql {all|2|4|6|8-10|11-14|16|18-19|all}
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    
    name VARCHAR(200) NOT NULL,   -- NOT NULL — обязательное поле
    
    sku VARCHAR(50) NOT NULL UNIQUE,     -- UNIQUE — уникальное значение
    
    -- CHECK — проверка условия
    price NUMERIC(10, 2) NOT NULL CHECK (price > 0),
    stock_quantity INTEGER DEFAULT 0 CHECK (stock_quantity >= 0),
    
    -- DEFAULT — значение по умолчанию
    status VARCHAR(20) DEFAULT 'active',
    is_featured BOOLEAN DEFAULT FALSE,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Именованные ограничения
    CONSTRAINT valid_status CHECK (status IN ('active', 'inactive', 'discontinued'))
);
```

---

# Где делать ограничения? В базе данных или в приложении?

**Лучший подход**: **И в приложении, и в базе данных**, но с разделением ответственности.

| Что проверять | Где лучше | Почему | Важность |
|---------------|-----------|--------|----------|
| NOT NULL | Только в базе | Защита от прямых INSERT | ★★★★★ |
| PRIMARY KEY / UNIQUE | Только в базе | Гарантия уникальности | ★★★★★ |
| FOREIGN KEY | Только в базе | Ссылочная целостность | ★★★★★ |
| Простые бизнес-правила | И в базе, и в приложении | База — защита, приложение — UX | ★★★★ |
| Сложная бизнес-логика | В основном в приложении | Много таблиц, внешние системы | ★★★ |

---

# Практическое правило 80/20

Где что проверять

<div class="grid grid-cols-2 gap-8">

<div>

### В базе данных ✅

**Всё, что обязательно должно быть всегда верно:**

- NOT NULL
- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- Простые CHECK

Это **последняя линия обороны**.

</div>

<div>

### В приложении ✅

**Всё важное для пользователя:**

- Форматирование
- Рекомендации
- Сложные бизнес-правила
- Контекстная валидация
- Мгновенная обратная связь

Это **первая линия** и лучший UX.

</div>

</div>

<v-click>

⚠️ **Важно**: Приложение даёт быстрый фидбек (50-400ms), база — гарантирует целостность даже при багах или прямом доступе.

</v-click>

---

# Первичные ключи (Primary Keys)

Уникальный идентификатор записи

**PRIMARY KEY** — один или несколько столбцов, которые уникально идентифицируют каждую строку.

#### **Первичный ключ и Именованный первичный ключ:** 

<br/>

```sql {all|1-5|7-11|all}
-- Простой первичный ключ
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL
);

-- Именованный первичный ключ
CREATE TABLE employees (
    emp_id INTEGER,
    CONSTRAINT pk_employees PRIMARY KEY (emp_id)
);
```

---

# Первичные ключи (Primary Keys)

```sql {all|1-6|7-12|all}
-- Составной первичный ключ
CREATE TABLE course_enrollments (
    student_id INTEGER,
    course_id INTEGER,
    PRIMARY KEY (student_id, course_id)
);

-- UUID как первичный ключ
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id INTEGER NOT NULL
);
```

---

# Внешние ключи (Foreign Keys)

Связи между таблицами

**FOREIGN KEY** — столбец, который ссылается на первичный ключ другой таблицы.

```sql {all|1-6|8-15|17-19|all}
-- Создание таблиц с внешними ключами
CREATE TABLE authors (
    author_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    bio TEXT
);

CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author_id INTEGER NOT NULL,
    
    -- Внешний ключ
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);

-- Попытка вставить книгу с несуществующим автором
INSERT INTO books (title, author_id) VALUES ('Книга', 999);
-- ERROR: violates foreign key constraint
```

---

# Каскадные операции

ON DELETE и ON UPDATE

**Каскадные операции** — автоматические действия при изменении/удалении родительской записи.

```sql {all|1-5|6-10|1-10|11-14|all}
CREATE TABLE book_reviews (
    review_id SERIAL PRIMARY KEY,
    book_id INTEGER NOT NULL,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    
    CONSTRAINT fk_book
        FOREIGN KEY (book_id) REFERENCES books(book_id)
        ON DELETE CASCADE           -- При удалении книги удалить отзывы
        ON UPDATE CASCADE           -- При обновлении ID обновить здесь
);

-- NO ACTION (по умолчанию)
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    ON DELETE NO ACTION,            -- Запретить удаление
```
---

# Каскадные операции

```sql{all|1-3|4-7|8-10|all}
-- RESTRICT (проверка сразу)
FOREIGN KEY (product_id) REFERENCES products(product_id)
    ON DELETE RESTRICT,

-- SET NULL
FOREIGN KEY (product_id) REFERENCES products(product_id)
    ON DELETE SET NULL,             -- Установить NULL

-- SET DEFAULT
ON DELETE SET DEFAULT               -- Установить значение по умолчанию
```

---

# Связь один-ко-многим

Пример: авторы и книги

```sql {all|1-5|7-11|13-15|17-20|all}
-- Один автор -> много книг
CREATE TABLE authors (
    author_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author_id INTEGER REFERENCES authors(author_id) ON DELETE CASCADE
);

-- Вставка данных
INSERT INTO authors (name) VALUES ('Лев Толстой');
INSERT INTO books (title, author_id) VALUES ('Война и мир', 1);

-- Один автор, много книг
SELECT a.name, b.title
FROM authors a
JOIN books b ON a.author_id = b.author_id;
```

---

# Связь многие-ко-многим

### Создание таблицы студентов и курсов

```sql {all|1-6|8-13|all}
-- Студенты
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- Курсы
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    credits INTEGER
);
```

---

# Связь многие-ко-многим


### Связующая таблица и запрос для получения списка курсов, которые посещает студент

```sql{all|1-7|9-13|all}
-- Связующая таблица (junction table)
CREATE TABLE student_courses (
    student_id INTEGER REFERENCES students(student_id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES courses(course_id) ON DELETE CASCADE,
    enrollment_date DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (student_id, course_id)
);

-- Запрос: какие курсы посещает студент
SELECT c.course_name
FROM student_courses sc
JOIN courses c ON sc.course_id = c.course_id
WHERE sc.student_id = 1;
```

---

# ALTER TABLE

Изменение структуры таблицы

```sql {all|1-2|4-5|7-8|10-11|13-14|16-17|19-20|all}
-- Добавление колонки
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Добавление с DEFAULT
ALTER TABLE users ADD COLUMN country VARCHAR(50) DEFAULT 'Russia';

-- Удаление колонки
ALTER TABLE users DROP COLUMN phone;

-- Переименование колонки
ALTER TABLE users RENAME COLUMN age TO user_age;

-- Изменение типа данных
ALTER TABLE users ALTER COLUMN username TYPE VARCHAR(100);

-- Установка NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- Установка значения по умолчанию
ALTER TABLE users ALTER COLUMN is_active SET DEFAULT TRUE;
```

---

# ALTER TABLE: ограничения

Добавление и удаление

```sql {all|1-3|5-7|9-10|12-13|all}
-- Добавление ограничения
ALTER TABLE users ADD CONSTRAINT check_username_length 
    CHECK (LENGTH(username) >= 3);

-- Удаление ограничения
ALTER TABLE users 
    DROP CONSTRAINT check_username_length;

-- Переименование таблицы
ALTER TABLE users RENAME TO app_users;

-- Изменение владельца
ALTER TABLE app_users OWNER TO new_owner;
```

---

# DROP и TRUNCATE

Удаление объектов и очистка таблиц

<div class="grid grid-cols-2 gap-8">

<div>

### DROP — удаление объектов

```sql{all}
-- Удаление таблицы
DROP TABLE products;

-- С проверкой существования
DROP TABLE IF EXISTS products;

-- Каскадное удаление
DROP TABLE products CASCADE;

-- Удаление БД
DROP DATABASE IF EXISTS old_db;
```

</div>

<div>

### TRUNCATE — очистка таблицы

```sql{all}
-- Удалить все данные
TRUNCATE TABLE logs;

-- С перезапуском SERIAL
TRUNCATE TABLE logs 
    RESTART IDENTITY;

-- Каскадная очистка
TRUNCATE TABLE orders CASCADE;

-- Несколько таблиц
TRUNCATE TABLE logs, sessions;
```

</div>

</div>

<v-click>

⚠️ **TRUNCATE** быстрее DELETE, но не может использовать WHERE и блокирует всю таблицу.

</v-click>

---

# TRUNCATE vs DELETE - Ключевые различия

| Параметр | DELETE | TRUNCATE | Победитель |
|----------|--------|----------|------------|
| Скорость (миллионы строк) | Медленно | Очень быстро | **TRUNCATE** ★★★★★ |
| WHERE-условие | Да | Нет | **DELETE** |
| Триггеры | Срабатывают | Не срабатывают | — |
| FOREIGN KEY | Работает всегда | Запрещено без CASCADE | **DELETE** |
| Блокировка | Мягче | AccessExclusiveLock | **DELETE** |
| Откат (rollback) | Да | Да  | Оба |
| Сброс SERIAL | Нет | Можно | **TRUNCATE** |

---

# Когда использовать TRUNCATE vs DELETE

Практические рекомендации

<div class="grid grid-cols-2 gap-8">

<div>

### Используй TRUNCATE когда:

- ✅ Нужно очистить всю таблицу
- ✅ Миллионы строк
- ✅ Временные/кеш-таблицы
- ✅ Нет важных триггеров
- ✅ Можешь позволить блокировку

```sql{all}
TRUNCATE TABLE cache_data 
    RESTART IDENTITY;
```

</div>

<div>

### Используй DELETE когда:

- ✅ Нужна фильтрация (WHERE)
- ✅ Есть FOREIGN KEY
- ✅ Важны триггеры
- ✅ Продакшен с нагрузкой
- ✅ Частичная очистка

```sql{all}
DELETE FROM logs 
WHERE created_at < NOW() - INTERVAL '30 days';
```

</div>

</div>

---

# INSERT — вставка данных

Базовый синтаксис

```sql {all|1-3|5-7|9-13|15-17|all}
-- Вставка одной строки
INSERT INTO users (username, email, first_name, last_name)
VALUES ('john_doe', 'john@example.com', 'John', 'Doe');

-- Вставка с автоматическим ID
INSERT INTO users (username, email)
VALUES ('jane_smith', 'jane@example.com');

-- Вставка нескольких строк
INSERT INTO users (username, email, first_name, last_name) VALUES
    ('alice', 'alice@example.com', 'Alice', 'Brown'),
    ('bob', 'bob@example.com', 'Bob', 'Wilson'),
    ('charlie', 'charlie@example.com', 'Charlie', 'Davis');

-- Вставка с подзапросом
INSERT INTO active_users (user_id, username, email)
SELECT user_id, username, email FROM users WHERE is_active = TRUE;
```

---

# INSERT RETURNING

Возврат вставленных данных

```sql {all|1-4|6-9|11-14|all}
-- Вернуть ID вставленной записи
INSERT INTO products (name, price)
VALUES ('Новый товар', 1999.99)
RETURNING product_id;

-- Вернуть все данные
INSERT INTO users (username, email)
VALUES ('new_user', 'new@example.com')
RETURNING *;

-- Вернуть вычисленные значения
INSERT INTO orders (customer_id, total_amount)
VALUES (1, 5000.00)
RETURNING order_id, total_amount * 1.2 AS with_tax;
```

<v-click>

✨ Очень полезно для получения сгенерированного ID без дополнительного запроса!

</v-click>

---

# ON CONFLICT (UPSERT)

Обработка конфликтов

```sql {all|1-4|6-11|13-18|all}
-- Игнорировать конфликт
INSERT INTO users (username, email)
VALUES ('existing_user', 'existing@example.com')
ON CONFLICT (username) DO NOTHING;

-- Обновить при конфликте
INSERT INTO products (sku, name, price)
VALUES ('SKU123', 'Product', 100.00)
ON CONFLICT (sku) DO UPDATE SET 
    name = EXCLUDED.name,
    price = EXCLUDED.price;

-- Условное обновление
INSERT INTO inventory (product_id, quantity)
VALUES (1, 100)
ON CONFLICT (product_id) DO UPDATE SET 
    quantity = inventory.quantity + EXCLUDED.quantity
WHERE inventory.quantity < 1000;
```

---

# SELECT — выборка данных

Базовая выборка

```sql {all|1-2|4-9|10-12|14-17|all}
-- Выбрать все столбцы
SELECT * FROM users;

-- Выбрать конкретные столбцы с алиасами
SELECT 
    user_id AS id,
    username AS login,
    email AS "E-mail"
FROM users;

-- DISTINCT — уникальные значения
SELECT DISTINCT country FROM users;

-- DISTINCT ON — первая строка для каждого значения
SELECT DISTINCT ON (country) country, city, username
FROM users
ORDER BY country, created_at DESC;
```

---

# WHERE — фильтрация

### Условия выборки: Простые, Логические, диапазоны, списки значений

```sql {all|1-4|6-9|10-13|13-16|all}
-- Простые условия
SELECT * FROM products WHERE price > 1000;
SELECT * FROM users WHERE is_active = TRUE;
SELECT * FROM products WHERE category = 'Electronics';

-- Логические операторы
SELECT * FROM products 
WHERE price > 1000 AND stock_quantity > 0;

-- Диапазоны
SELECT * FROM products 
WHERE price BETWEEN 1000 AND 5000;

-- Списки значений
SELECT * FROM users 
WHERE country IN ('Russia', 'Belarus', 'Kazakhstan');
```
---

# WHERE — фильтрация

### Условия выборки: Поиск по шаблону, регистронезависимые, NULL проверки

```sql {all|1-4|5-7|8-10|all}
-- Поиск по шаблону
SELECT * FROM users WHERE username LIKE 'john%';  -- Начинается с
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- Заканчивается на
SELECT * FROM users WHERE username LIKE 'j_hn';  -- _ = один символ

-- ILIKE — регистронезависимый
SELECT * FROM users WHERE email ILIKE '%GMAIL.COM';

-- NULL проверки
SELECT * FROM users WHERE phone IS NULL;
```

---

# ORDER BY и LIMIT

Сортировка и ограничение результатов

```sql {all|1-3|5-6|8-10|12-14|16-20|all}
-- Сортировка по возрастанию
SELECT * FROM products ORDER BY price;
SELECT * FROM products ORDER BY price ASC;  -- Явно

-- Сортировка по убыванию
SELECT * FROM products ORDER BY price DESC;

-- Множественная сортировка
SELECT * FROM products 
ORDER BY category ASC, price DESC;

-- NULL в начале/конце
SELECT * FROM products ORDER BY discount_price NULLS FIRST;
SELECT * FROM products ORDER BY discount_price NULLS LAST;

-- LIMIT и OFFSET (пагинация)
SELECT * FROM products LIMIT 10;                    -- Первые 10
SELECT * FROM products LIMIT 10 OFFSET 10;          -- Записи 11-20
SELECT * FROM products 
LIMIT 20 OFFSET 40;  -- Страница 3 по 20 записей
```

---

# Пагинация

Разделение данных на страницы

**Пагинация** — способ разделения большого количества данных на страницы фиксированного размера.

```sql {all|1-4|6-9|all}
-- Страница 1 (записи 1-20)
SELECT * FROM products 
ORDER BY product_id
LIMIT 20 OFFSET 0;

-- Страница 2 (записи 21-40)
SELECT * FROM products 
ORDER BY product_id
LIMIT 20 OFFSET 20;

-- Страница N
-- OFFSET = (страница - 1) * размер_страницы
```

<v-click>

💡 **Всегда используйте ORDER BY** для консистентности результатов!

</v-click>

---

# Агрегатные функции

Вычисления над наборами строк

**Агрегатные функции** выполняют вычисление над набором строк и возвращают одно значение.

### Стандартные агрегатные функции: Count, SUM, AVG

```sql {all|1-3|5-7|9-10|all}
-- COUNT — подсчет
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT country) FROM users;

-- SUM — сумма
SELECT SUM(price) FROM products;
SELECT SUM(quantity * price) FROM order_items;

-- AVG — среднее
SELECT AVG(price) FROM products;
```
---

# Агрегатные функции

### Стандартные агрегатные функции: Min, Max

```sql {all|1-2|4-11|all}
-- MIN / MAX — минимум/максимум
SELECT MIN(price), MAX(price) FROM products;

-- Несколько агрегатов
SELECT 
    COUNT(*) AS total_products,
    SUM(stock_quantity) AS total_stock,
    AVG(price) AS avg_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products;
```

---

# GROUP BY — группировка

Группировка строк для агрегации

**GROUP BY** группирует строки по одинаковым значениям и применяет агрегатные функции к каждой группе.

```sql {all|1-5|7-13|all}
-- Количество пользователей по странам
SELECT country, COUNT(*) AS user_count
FROM users
GROUP BY country
ORDER BY user_count DESC;

-- Средняя цена по категориям
SELECT 
    category,
    AVG(price) AS avg_price,
    COUNT(*) AS product_count
FROM products
GROUP BY category;
```

---

# GROUP BY — группировка

### Группировка по нескольким колонкам

```sql{all}
SELECT 
    category,
    EXTRACT(YEAR FROM created_at) AS year,
    COUNT(*) AS count
FROM products
GROUP BY category, EXTRACT(YEAR FROM created_at)
ORDER BY category, year;
```

---

# HAVING — фильтрация групп

Условия после группировки

**HAVING** фильтрует группы после `GROUP BY` и вычисления агрегатов.

```sql {all|1-5|7-17|all}
-- Категории с более чем 10 товарами
SELECT category, COUNT(*) AS count
FROM products
GROUP BY category
HAVING COUNT(*) > 10;

-- Клиенты с суммой заказов > 10000
SELECT 
    customer_id,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(total_amount) > 10000
ORDER BY total_spent DESC;
```

---

# HAVING — фильтрация групп

### WHERE и HAVING вместе

```sql{all}
SELECT 
    category,
    AVG(price) AS avg_price
FROM products
WHERE is_active = TRUE           -- Фильтр ДО группировки
GROUP BY category
HAVING AVG(price) > 1000         -- Фильтр ПОСЛЕ группировки
```

<v-click>

⚠️ **WHERE** фильтрует строки, **HAVING** фильтрует группы!

</v-click>

---

# UPDATE — обновление данных

Изменение существующих записей

```sql {all|1-4|6-12|all}
-- Обновление одного поля
UPDATE users 
SET email = 'newemail@example.com'
WHERE user_id = 1;

-- Обновление нескольких полей
UPDATE products 
SET 
    price = 1999.99,
    stock_quantity = 100,
    updated_at = NOW()
WHERE product_id = 5;
```

---

# UPDATE — обновление данных


```sql {all|1-4|6-10|all}
-- Обновление с вычислением
UPDATE products 
SET price = price * 1.1  -- Увеличить на 10%
WHERE category = 'Electronics';

-- UPDATE с RETURNING
UPDATE products 
SET price = price * 0.9
WHERE category = 'Clearance'
RETURNING product_id, name, price AS new_price;
```

<v-click>

⚠️ **Всегда используйте WHERE!** Без него обновятся ВСЕ записи.

</v-click>

---

# DELETE — удаление данных

Удаление записей из таблицы

```sql {all|1-2|4-5|7-9|10-15|16-20|all}
-- Удаление одной записи
DELETE FROM users WHERE user_id = 1;

-- Удаление по условию
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days';

-- Удаление с подзапросом
DELETE FROM orders 
WHERE customer_id IN (SELECT customer_id FROM customers WHERE status = 'deleted');

-- DELETE с RETURNING
DELETE FROM sessions 
WHERE expires_at < NOW()
RETURNING session_id, user_id;

-- Удаление дубликатов
DELETE FROM products p1
USING products p2
WHERE p1.sku = p2.sku 
  AND p1.product_id > p2.product_id;
```

<v-click>

⚠️ **DELETE без WHERE удалит ВСЕ записи!** Сначала проверьте SELECT.

</v-click>

---

# Подзапросы

Запросы внутри запросов

### Скалярные подзапросы (одно значение)

```sql {all|1-6|8-10|all}
-- В SELECT
SELECT 
    name,
    price,
    (SELECT AVG(price) FROM products) AS avg_price
FROM products;

-- В WHERE
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

---

# IN, EXISTS

Подзапросы в условиях

```sql {all|1-6|7-13|all}
-- IN
SELECT name, email
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id FROM orders
);

-- EXISTS (часто быстрее IN)
SELECT name, email
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

# NOT EXISTS, ANY/ALL


```sql {all|1-6|8-14|all}
-- NOT EXISTS
SELECT name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM reviews r WHERE r.product_id = p.product_id
);

-- ANY и ALL
SELECT name, price
FROM products
WHERE price > ALL (
    SELECT price FROM products WHERE category = 'Books'
);
```

---

# UNION, INTERSECT, EXCEPT

Операции с множествами

```sql {all|1-4|6-9|11-15|16-21|all}
-- UNION — объединение (без дубликатов)
SELECT name FROM customers
UNION
SELECT name FROM suppliers;

-- UNION ALL — с дубликатами (быстрее)
SELECT email FROM users
UNION ALL
SELECT email FROM subscribers;

-- INTERSECT — пересечение
SELECT email FROM customers
INTERSECT
SELECT email FROM suppliers;

-- EXCEPT — разность
SELECT user_id FROM users
EXCEPT
SELECT customer_id FROM customers;
```

---

# CASE — условные выражения

SQL версия if-else

```sql {5-9|all}
-- Простой CASE
SELECT 
    name,
    price,
    CASE 
        WHEN price < 1000 THEN 'Дешевый'
        WHEN price < 5000 THEN 'Средний'
        ELSE 'Дорогой'
    END AS price_category
FROM products;
```
---

# CASE в агрегации и в Update

```sql {5|11-17|all}
-- В агрегации
SELECT 
    category,
    COUNT(*) AS total,
    COUNT(CASE WHEN price > 1000 THEN 1 END) AS expensive_count
FROM products
GROUP BY category;

-- В UPDATE
UPDATE products
SET discount = CASE
    WHEN price > 10000 THEN 0.15
    WHEN price > 5000 THEN 0.10
    WHEN price > 1000 THEN 0.05
    ELSE 0
END;
```

---

# COALESCE и NULLIF

Работа с NULL

<div class="grid grid-cols-2 gap-8">

<div>

### COALESCE

Вернуть первое NOT NULL

```sql{all|1-6|8-14|all}
-- Первое NOT NULL значение
SELECT 
    name,
    COALESCE(phone, mobile, 'No contact') 
        AS contact
FROM users;

-- Замена NULL в агрегации
SELECT 
    category,
    COALESCE(SUM(stock), 0) 
        AS total_stock
FROM products
GROUP BY category;
```

</div>

<div>

### NULLIF

Преобразовать в NULL

```sql{all|1-6|8-13|all}
-- NULL если значения равны
SELECT 
    name,
    NULLIF(discount_price, 0) 
        AS discount
FROM products;

-- Избежание деления на 0
SELECT 
    product_id,
    returns / NULLIF(total_sales, 0) 
        AS return_rate
FROM stats;
```

</div>

</div>

---

# Функции работы со строками

### Конкатенация + Изменение регистра 

```sql {all|1-5|7-12|all}
-- Конкатенация
SELECT 
    first_name || ' ' || last_name AS full_name,
    CONCAT_WS(' ', first_name, middle_name, last_name) AS full
FROM users;

-- Изменение регистра
SELECT 
    UPPER(name) AS uppercase,
    LOWER(name) AS lowercase,
    INITCAP(name) AS titlecase
FROM products;
```

---

# Функции работы со строками

### Обрезка и замена + Подстроки и длина

```sql{1-4|6-10|all}
-- Обрезка и замена
SELECT 
    TRIM('  text  ') AS trimmed,
    REPLACE('hello world', 'world', 'PostgreSQL') AS replaced;

-- Подстроки и длина
SELECT 
    SUBSTRING(description FROM 1 FOR 100) AS short_desc,
    LENGTH(description) AS char_count
FROM articles;
```

---

# Функции работы с датами

### Текущие дата и время + Извлечение компонентов

```sql {all|1-6|8-15|all}
-- Текущие дата и время
SELECT 
    CURRENT_DATE AS today,
    CURRENT_TIME AS now_time,
    CURRENT_TIMESTAMP AS now,
    NOW() AS now_func;

-- Извлечение компонентов
SELECT 
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    EXTRACT(DAY FROM created_at) AS day,
    EXTRACT(DOW FROM created_at) AS day_of_week,
    EXTRACT(EPOCH FROM created_at) AS unix_timestamp
FROM orders;
```

---

# Функции работы с датами

### Арифметика с датами + Округление дат

```sql{1-8|9-14|all}
-- Арифметика с датами
SELECT 
    order_date,
    order_date + INTERVAL '7 days' AS week_later,
    order_date - INTERVAL '1 month' AS month_ago,
    NOW() - order_date AS time_since_order,
    AGE(NOW(), order_date) AS age
FROM orders;

-- Округление дат
SELECT 
    DATE_TRUNC('day', created_at) AS day_start,
    DATE_TRUNC('month', created_at) AS month_start
FROM logs;
```

---

# Полезные ресурсы

Где учиться и практиковаться

### Практика SQL

- 🎯 **pgexercises.com** — интерактивные упражнения
- 🎓 **sqlzoo.net** — обучение с примерами
- 💻 **hackerrank.com/domains/sql** — задачи

### Документация

- 📚 **postgresql.org/docs** — официальная документация
- 📖 **postgresql.org/docs/tutorial** — туториалы

### Инструменты

- 🔍 **explain.depesz.com** — визуализация EXPLAIN
- ⚡ **db-fiddle.com** — онлайн SQL песочница

---

# Контрольные вопросы

Проверьте свои знания

1. В чем разница между DDL и DML?
2. Какие каскадные операции доступны для внешних ключей?
3. Когда использовать TRUNCATE вместо DELETE?
4. В чем разница между INNER JOIN и LEFT JOIN?
5. Когда использовать EXISTS вместо IN?
6. Какие уровни изоляции транзакций существуют?
7. В чем разница между UNION и UNION ALL?
8. Как работает ON CONFLICT в INSERT?
9. Зачем нужны индексы и когда их не создавать?
10. Что такое ACID свойства транзакций?

---
layout: center
class: text-center
---

# Ключевые выводы

Что запомнить

<div class="grid grid-cols-2 gap-8 mt-12">

<div>

### ✅ Делайте

- Используйте TIMESTAMPTZ
- Всегда WHERE в UPDATE/DELETE
- Создавайте индексы на FK
- Параметризованные запросы
- Транзакции для критичных операций
- EXISTS вместо IN

</div>

<div>

### ❌ Избегайте

- SELECT * без необходимости
- Функции на индексах в WHERE
- Конкатенация SQL строк
- DELETE/UPDATE без WHERE
- Избыточные индексы
- CROSS JOIN на больших таблицах

</div>

</div>

<div class="mt-12">

> **SQL — это не просто язык запросов, это способ мышления о данных**

</div>

---
layout: end
class: text-center
---

# Спасибо за внимание!

Практикуйтесь и изучайте PostgreSQL

<div class="mt-8">
  <a href="https://postgresql.org" target="_blank" class="text-blue-500 hover:underline">
    postgresql.org — официальная документация
  </a>
</div>

<div class="mt-4">
  <a href="https://pgexercises.com" target="_blank" class="text-blue-500 hover:underline">
    pgexercises.com — практические упражнения
  </a>
</div>
