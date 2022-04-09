# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.
```
version: '3.5'

services:
  postgres:
    container_name: postgres_container
    image: postgres:13.5
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-Temp001}
      PGDATA: /data/postgres
    volumes:
       - postgres:/data/postgres
    ports:
      - "5433:5432"
    networks:
      - postgres
    restart: unless-stopped
  
networks:
  postgres:
    driver: bridge

volumes:
  postgres: 
```
Подключитесь к БД PostgreSQL используя `psql`.
```
root@a592f032153c:/# psql --version
psql (PostgreSQL) 13.5 (Debian 13.5-1.pgdg110+1)
```
Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
```
postgres-# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
```
- подключения к БД
```
postgres=# \c
You are now connected to database "postgres" as user "postgres".
```
- вывода списка таблиц
```  
  postgres=# \dt
```
- вывода описания содержимого таблиц
```  
  postgres=# \dt+
```
- выхода из psql
```  
  postgres=# \q
```

## Задача 2

Используя `psql` создайте БД `test_database`.
```
create database test_database;
```
Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).
Восстановите бэкап БД в `test_database`.
Перейдите в управляющую консоль `psql` внутри контейнера.
Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
```
docker cp test_dump.sql a592f032153c:/var/lib/postgresql/data_backup/   test_dump.sql
turganovai@ch-32:~/gitlab/cg024_postgre_edu$ docker exec -it a592f032153c  bash
psql -U postgres -d test_database < /var/lib/postgresql/data_backup/test_dump.sql
postgres-# \c test_database 
You are now connected to database "test_database" as user "postgres".

test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

test_database=# EXPLAIN ANALYZE select * from orders;
                                           QUERY PLAN                                            
-------------------------------------------------------------------------------------------------
 Seq Scan on orders  (cost=0.00..1.08 rows=8 width=24) (actual time=0.006..0.008 rows=8 loops=1)
 Planning Time: 0.134 ms
 Execution Time: 0.022 ms
(3 rows)
```

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.
```
test_database=# SELECT attname, avg_width from pg_stats WHERE tablename='orders';
 attname | avg_width 
---------+-----------
 id      |         4
 title   |        16
 price   |         4
 
 
ответ: столбец title
```
**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

```
test_database=# CREATE TABLE orders1 (CHECK ( price > 499 )) INHERITS (orders);
CREATE TABLE
test_database=# CREATE TABLE orders2 (CHECK ( price <= 499 )) INHERITS (orders);
CREATE TABLE
```

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?
```
Как вариант провести партицирование.
```
## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.
```buildoutcfg
root@a592f032153c:~# pg_dump -U postgres test_database -f /var/lib/postgresql/test_dump2.sql
```
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?
```buildoutcfg
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0,
    -- Добавить ключ уникальности
    CONSTRAINT title_unique UNIQUE (title)
);
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
