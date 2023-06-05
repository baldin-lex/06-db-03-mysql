# Домашнее задание к занятию 3. «MySQL» - Балдин

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Решение:

1. *С использованием [docker-compose.yaml](docker-compose.yaml) поднимаю инстанс MySQL*

2. *Восстанавливаюсь из бэкапа:*
```bash
[baldin@localhost ~]$ sudo docker cp test_dump.sql mysql:/tmp
[sudo] пароль для baldin: 
Successfully copied 4.1kB to mysql:/tmp
[baldin@localhost ~]$ sudo docker exec -it mysql bash
bash-4.4# ls /tmp
test_dump.sql
bash-4.4# mysql -u root -p test_db < /tmp/test_dump.sql
Enter password:
```
3. *Перехожу в управляющую консоль*
```bash
bash-4.4# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.33 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
4. *Проверяю статус сервера БД при помощи команды `\s`*
```bash
mysql> \s
--------------
mysql  Ver 8.0.33 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:		9
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.33 MySQL Community Server - GPL
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8mb4
Db     characterset:	utf8mb4
Client characterset:	latin1
Conn.  characterset:	latin1
UNIX socket:		/var/run/mysqld/mysqld.sock
Binary data as:		Hexadecimal
Uptime:			6 min 17 sec

Threads: 2  Questions: 35  Slow queries: 0  Opens: 138  Flush tables: 3  Open tables: 56  Queries per second avg: 0.092
--------------
```
5. *Вывожу список таблиц из восстановленной БД*
```bash
mysql> USE test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)
```
6. *Вывод количества записей с `price > 300`*
```bash
mysql> SELECT COUNT(*) FROM orders WHERE price > '300';
+----------+
| COUNT(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)
```
---
## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

## Решение:

1. *Завел пользователя с указанными параметрами и предоставил привелегии на SELECT*
```bash
mysql> CREATE USER 'test' IDENTIFIED WITH mysql_native_password BY 'test-pass'
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3
    -> PASSWORD_LOCK_TIME 1
    -> ATTRIBUTE '{"lastname": "Pretty", "firstname": "James"}'
    -> ;
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT SELECT ON `test_db`.* TO 'test';
Query OK, 0 rows affected (0.01 sec)
```
2. *Данные по пользователю `test`*
```bash
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+------+----------------------------------------------+
| USER | HOST | ATTRIBUTE                                    |
+------+------+----------------------------------------------+
| test | %    | {"lastname": "Pretty", "firstname": "James"} |
+------+------+----------------------------------------------+
1 row in set (0.00 sec)
```
---
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

## Решение:

1. *Устанавливаю профилирование `SET profiling = 1`*
```bash
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```
2. *Ниже в выводе видно, что используется InnoDB*
```bash

mysql> SELECT table_schema,table_name,engine FROM information_schema.tables WHERE table_schema = DATABASE();
+--------------+------------+--------+
| TABLE_SCHEMA | TABLE_NAME | ENGINE |
+--------------+------------+--------+
| test_db      | orders     | InnoDB |
+--------------+------------+--------+
1 row in set (0.02 sec)
```
3. *Меняю на 'MyISAM', потом снова на 'InnoDB' и смотрю время выполнения*
```bash
mysql> ALTER table orders engine = 'MyISAM';
Query OK, 5 rows affected (0.09 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> ALTER table orders engine = 'InnoDB';
Query OK, 5 rows affected (0.16 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show profiles;
+----------+------------+------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                |
+----------+------------+------------------------------------------------------------------------------------------------------+
|        1 | 0.01345625 | SELECT table_schema,table_name,engine FROM information_schema.tables WHERE table_schema = DATABASE() |
|        2 | 0.18240000 | ALTER TABLE orders ENGINE = InnoDB                                                                   |
|        3 | 0.13719550 | alter table orders engine = 'MyISAM'                                                                 |
|        4 | 0.10437950 | ALTER TABLE orders ENGINE = InnoDB                                                                   |
|        5 | 0.09282300 | alter table orders engine = 'MyISAM'                                                                 |
|        6 | 0.15983150 | alter table orders engine = 'InnoDB'                                                                 |
+----------+------------+------------------------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```
---
## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

## Решение:

*Мой файл `my.cnf` находился в директории /etc. Я так понимаю, для выполнения условий ТЗ его нужно привести к следующему виду:*
```bash
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/

innodb_flush_log_at_trx_commit = 0 # или 2 здесь уместнее
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 1840M
innodb_log_file_size = 100M
```
