# 📘 PostgreSQL & PL/pgSQL Exercises

This repository contains solutions to Exercises 1–9 using PostgreSQL stored functions and procedures in PL/pgSQL.

---

## ✅ Exercise 1 — Greeting with Name and Timestamp

```sql
DROP FUNCTION IF EXISTS greet_user(TEXT);

CREATE OR REPLACE FUNCTION greet_user(name TEXT) 
RETURNS VARCHAR
LANGUAGE plpgsql AS
$$
BEGIN
    RETURN CONCAT('hello', name, current_timestamp);
END;
$$;

SELECT greet_user('karin');
```

---

## ✅ Exercise 2 — Create Table for Orders

```sql
DROP PROCEDURE IF EXISTS create_orders_table;

CREATE OR REPLACE PROCEDURE create_orders_table()
LANGUAGE plpgsql AS
$$
BEGIN
    CREATE TABLE IF NOT EXISTS orders (
        id SERIAL PRIMARY KEY,
        customer_name TEXT NOT NULL,
        amount DOUBLE PRECISION NOT NULL
    );
END;
$$;

CALL create_orders_table();
```

---

## ✅ Exercise 3 — Multiply Three Numbers

```sql
CREATE OR REPLACE FUNCTION multiply_three(
    x DOUBLE PRECISION,
    y DOUBLE PRECISION,
    z DOUBLE PRECISION)
RETURNS DOUBLE PRECISION
LANGUAGE plpgsql AS
$$
BEGIN
    RETURN x * y * z;
END;
$$;

SELECT multiply_three(2, 3, 4);
```

---

## ✅ Exercise 4 — Division and Modulo Function

```sql
CREATE OR REPLACE FUNCTION div_mod(
    a DOUBLE PRECISION,
    b DOUBLE PRECISION,
    OUT quotient DOUBLE PRECISION,
    OUT remainder INTEGER
)
LANGUAGE plpgsql AS
$$
BEGIN
    quotient := a / b;
    remainder := CAST(a AS INTEGER) % CAST(b AS INTEGER);
END;
$$;

SELECT * FROM div_mod(17, 5);
```

---

## ✅ Exercise 5 — Square Root and Power 4

```sql
CREATE OR REPLACE FUNCTION sp_math_roots(
    x DOUBLE PRECISION,
    y DOUBLE PRECISION,
    OUT sum_result DOUBLE PRECISION,
    OUT diff_result DOUBLE PRECISION,
    OUT x_squared DOUBLE PRECISION,
    OUT y_power_4 DOUBLE PRECISION
)
LANGUAGE plpgsql AS
$$
BEGIN
    sum_result := x + y;
    diff_result := x - y;
    x_squared := SQRT(x);
    y_power_4 := POWER(y, 4);
END;
$$;
```

---

## ✅ Exercise 6 — Insert Books and Get Books with Year

```sql
DROP PROCEDURE IF EXISTS prepare_books_db;

CREATE OR REPLACE PROCEDURE prepare_books_db()
LANGUAGE plpgsql AS
$$
BEGIN
    CREATE TABLE IF NOT EXISTS authors (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL
    );

    CREATE TABLE IF NOT EXISTS books (
        id SERIAL PRIMARY KEY,
        title TEXT,
        price DOUBLE PRECISION,
        publish_date DATE,
        author_id INT REFERENCES authors(id)
    );

    INSERT INTO authors(name) VALUES 
        ('Alice Munro'),
        ('George Orwell'),
        ('Haruki Murakami'),
        ('Chimamanda Ngozi Adichie');

    INSERT INTO books(title, price, publish_date, author_id) VALUES
        ('Lives of Girls and Women', 45.0, '1971-05-01', 1),
        ('1984', 30.0, '1949-06-08', 2),
        ('Norwegian Wood', 50.0, '1987-09-04', 3),
        ('Half of a Yellow Sun', 42.5, '2006-08-15', 4),
        ('Kafka on the Shore', 55.0, '2002-01-01', 3),
        ('Dear Life', 48.0, '2012-11-13', 1),
        ('The Thing Around Your Neck', 35.0, '2009-04-01', 4),
        ('Animal Farm', 28.0, '1945-08-17', 2),
        ('The Testaments', 60.0, '2019-09-10', 2),
        ('Colorless Tsukuru Tazaki', 47.5, '2013-04-12', 3);
END;
$$;

CALL prepare_books_db();
```

```sql
DROP FUNCTION IF EXISTS sp_get_books_with_year;

CREATE OR REPLACE FUNCTION sp_get_books_with_year()
RETURNS TABLE (
    title TEXT,
    publish_year INT,
    price DOUBLE PRECISION
)
LANGUAGE plpgsql AS
$$
BEGIN
    RETURN QUERY
    SELECT 
        title,
        EXTRACT(YEAR FROM publish_date)::INT,
        price
    FROM books;
END;
$$;

SELECT * FROM sp_get_books_with_year();
```

---

## ✅ Exercise 7 — Most Recently Published Book

```sql
DROP FUNCTION IF EXISTS sp_latest_book;

CREATE OR REPLACE FUNCTION sp_latest_book()
RETURNS TABLE (
    title TEXT,
    publish_date DATE
)
LANGUAGE plpgsql AS
$$
BEGIN
    RETURN QUERY
    SELECT title, publish_date
    FROM books
    ORDER BY publish_date DESC
    LIMIT 1;
END;
$$;

SELECT * FROM sp_latest_book();
```

---

## ✅ Exercise 8 — Books Summary Stats

```sql
DROP FUNCTION IF EXISTS sp_books_summary;

CREATE OR REPLACE FUNCTION sp_books_summary(
    OUT youngest_book DATE,
    OUT oldest_book DATE,
    OUT avg_price NUMERIC(5,2),
    OUT total_books INT
)
LANGUAGE plpgsql AS
$$
BEGIN
    SELECT 
        MAX(publish_date),
        MIN(publish_date),
        ROUND(AVG(price), 2),
        COUNT(*)
    INTO 
        youngest_book,
        oldest_book,
        avg_price,
        total_books
    FROM books;
END;
$$;

SELECT * FROM sp_books_summary();
```

---

## ✅ Exercise 9 — Books by Year Range

```sql
DROP FUNCTION IF EXISTS sp_books_by_year_range;

CREATE OR REPLACE FUNCTION sp_books_by_year_range(
    from_year INT,
    to_year INT
)
RETURNS TABLE (
    id INT,
    title TEXT,
    publish_date DATE,
    price DOUBLE PRECISION
)
LANGUAGE plpgsql AS
$$
BEGIN
    RETURN QUERY
    SELECT id, title, publish_date, price
    FROM books
    WHERE EXTRACT(YEAR FROM publish_date) BETWEEN from_year AND to_year;
END;
$$;

SELECT * FROM sp_books_by_year_range(2000, 2015);
```