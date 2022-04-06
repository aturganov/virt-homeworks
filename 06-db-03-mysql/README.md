
# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.
```
# docker-compose
version: '3.5'
services:
  mysql1:
    image: mysql:8
    restart: always
    environment:
      MYSQL_USER: locadm
      MYSQL_ROOT_PASSWORD: Temp001
      MYSQL_DATABASE: test_db
    # user: "1001:1001"
    user: root
    ports:
      - "3306:3306"
    volumes:
      - /home/turganovai/gitlab/cg023_mysql_edu/db:/var/lib/mysql
```
Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

# Backup
docker exec CONTAINER /usr/bin/mysqldump -u root --password=root DATABASE > backup.sql

# Restore
cat backup.sql | docker exec -i CONTAINER /usr/bin/mysql -u root --password=root DATABASE
Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.
```
Server version:         8.0.23 MySQL Community Server - GPL
```
Версия БД из статуса ниже:
```
mysql> status
--------------
mysql  Ver 8.0.23 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          17
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.23 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 2 hours 55 min 47 sec

Threads: 2  Questions: 38  Slow queries: 0  Opens: 183  Flush tables: 3  Open tables: 102  Queries per second avg: 0.003
```
Подключитесь к восстановленной БД и получите список таблиц из этой БД.
```
Не требовалось. Строчка по бэккапу
docker exec 34391bbba667 /usr/bin/mysqldump -u root --password=Temp001 test_db > test_db_backup.sql
Восстановление базы по заданию
mysql> create database test_dump;
cat test_dump.sql | docker exec -i 34391bbba667 /usr/bin/mysql -u root --password=Temp001
mysql> use test_dump
mysql> show tables;
+---------------------+
| Tables_in_test_dump |
+---------------------+
| orders              |
+---------------------+
1 row in set (0.00 sec)
```

**Приведите в ответе** количество записей с `price` > 300.
```
mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
Кол-во записей
mysql> select count(*) from orders where price > 300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)
```
В следующих заданиях мы будем продолжать работу с данным контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
```
CREATE USER 'test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'test-pass';
```
- срок истечения пароля - 180 дней 
```
mysql> ALTER USER test@'localhost' PASSWORD EXPIRE INTERVAL 180 DAY;
```
- количество попыток авторизации - 3
```
mysql> ALTER USER 'test'@'localhost' FAILED_LOGIN_ATTEMPTS 3;
```
- максимальное количество запросов в час - 100
```
ALTER USER 'test'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;
```
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"
```
mysql> ALTER USER 'test'@'localhost' ATTRIBUTE '{"lastName": "Pretty", "firstName": "James"}';
```

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
```
mysql> GRANT SELECT ON test_dump.orders TO 'test'@'localhost';
```
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.
```
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user = 'test';
+------+-----------+----------------------------------------------+
| USER | HOST      | ATTRIBUTE                                    |
+------+-----------+----------------------------------------------+
| test | localhost | {"lastName": "Pretty", "firstName": "James"} |
+------+-----------+----------------------------------------------+
1 row in set (0.00 sec)
```
## Задача 3

Установите профилирование `SET profiling = 1`.
```
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
Изучите вывод профилирования команд `SHOW PROFILES;`.
```
mysql> show profiles;
+----------+------------+-------------------+
| Query_ID | Duration   | Query             |
+----------+------------+-------------------+
|        1 | 0.00018175 | SET profiling = 1 |
+----------+------------+-------------------+
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user = 'test';
+------+-----------+----------------------------------------------+
| USER | HOST      | ATTRIBUTE                                    |
+------+-----------+----------------------------------------------+
| test | localhost | {"lastName": "Pretty", "firstName": "James"} |
+------+-----------+----------------------------------------------+
1 row in set (0.00 sec)

mysql> show profiles;
+----------+------------+----------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                |
+----------+------------+----------------------------------------------------------------------+
|        1 | 0.00018175 | SET profiling = 1                                                    |
|        2 | 0.00014800 | show procceslist                                                     |
|        3 | 0.00019925 | show processlist                                                     |
|        4 | 0.00083075 | select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user = 'test' |
```

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.
```
mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where table_name = 'orders';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
```
Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
```
mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.02 sec)
Records: 5  Duplicates: 0  Warnings: 0
```
- на `InnoDB`
```
mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0
```
```angular2html
mysql> show profiles;
+----------+------------+---------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                       |
+----------+------------+---------------------------------------------------------------------------------------------+
|        1 | 0.00018175 | SET profiling = 1                                                                           |
|        2 | 0.00014800 | show procceslist                                                                            |
|        3 | 0.00019925 | show processlist                                                                            |
|        4 | 0.00083075 | select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user = 'test'                        |
|        5 | 0.00766300 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES                                    |
|        6 | 0.00191025 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where table_name = 'orders'        |
|        7 | 0.00190125 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where table_name = 'orders'        |
|        8 | 0.00011300 | mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where table_name = 'orders' |
|        9 | 0.00084325 | ALTER TABLE ORDERS  ENGINE = MyISAM                                                         |
|       10 | 0.01603175 | ALTER TABLE orders ENGINE = MyISAM                                                          |
|       11 | 0.01431250 | ALTER TABLE orders ENGINE = InnoDB                                                          |
+----------+------------+---------------------------------------------------------------------------------------------+
11 rows in set, 1 warning (0.00 sec)
```
## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.
```
root@34391bbba667:/# cat  /etc/mysql/my.cnf 
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
```
Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.
```
Рекомендации взял тут
https://highload.today/index-php-2009-04-23-optimalnaya-nastroyka-mysql-servera/
root@34391bbba667:/# cat  /etc/mysql/my.cnf

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL


innodb_flush_log_at_trx_commit = 0 

# Хранение таблиц в отдельных файлах с компрессией
innodb_file_per_table = ON
innodb_file_format=Barracuda

innodb_log_buffer_size = 1M

# 30% от ОЗУ 251 * ,3 ~ 75 ГБ
# для данного сервака мягко говоря иррационально)
key_buffer_size = 76800M

max_binlog_size = 100M

# Custom config should go here
!includedir /etc/mysql/conf.d/
---
```
### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
