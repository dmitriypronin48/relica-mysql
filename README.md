Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - Пронин Дмитрий Андреевич



## Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

master-slave - один мастер и несколько слейв нод, данные копируются с ноды мастера. Из минусов если мастер выйдет из строя, то реплики не могут продолжить запись. Точнее могут, но данные при включении мастера не будут перезаписаны на мастер, а останутся в репликах.

master-master - каждый мастер реплицирует данные между мастерами, создаются взаимные зависимости. Из минусов не смогу назвать не чего, скорее всего это сложность конфигурации такого подхода.

## Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

На мастер ноде ставим настройку в mysql.conf.d/mysqld.cnf
```
server-id = 1
log-bin = mysql-bin
log-bin-index = mysql-bin.index
log-error = mysql-bin.err
relay-log = relay-bin
relay-log-info-file = relay-bin.info
relay-log-index = relay-bin.index
expire_logs_days=2

```
Рестартим сервис БД.

Для реплики в mysql.conf.d/mysqld.cnf

```
server-id = 2
relay-log = relay-bin
relay-log-info-file = relay-log.info
relay-log-index = relay-log.index
```
рестартим сервис.

В мастер ноде правалимаемся в mysql и делаем ряд комманд
```
use mysql;
CREATE USER 'replica_user'@'%' IDENTIFIED WITH mysql_native_password BY 'qwerty99';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
flush privileges;
SHOW MASTER STATUS;
```

Далее топаем на реплику:
```
CHANGE MASTER TO MASTER_HOST='<ip_master_сервера>', MASTER_USER='replica_user', MASTER_PASSWORD='qwerty99', MASTER_LOG_FILE = 'mysql-bin.000003', MASTER_LOG_POS = 0;
- mysql-bin.000003 - берем с мастера через команду SHOW MASTER STATUS;

START SLAVE;
SHOW SLAVE STATUS;
```

Скрин до создания таблицы на мастере
![скрин](https://github.com/dmitriypronin48/relica-mysql/blob/main/img/z1-1.jpg)


Скрин до создания таблицы на реплике
![скрин](https://github.com/dmitriypronin48/relica-mysql/blob/main/img/z1-2.jpg)


Дальше делаем тестовую таблицу на местере:

```
CREATE DATABASE only_replica_tests;
USE only_replica_tests;
CREATE TABLE test_replica (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Создал таблицу на мастере
![скрин](https://github.com/dmitriypronin48/relica-mysql/blob/main/img/z1-3.jpg)


Проверяю, что она создалась автоматом и на слейве, так как реплицируется

![скрин](https://github.com/dmitriypronin48/relica-mysql/blob/main/img/z1-4.jpg)


SHOW SLAVE STATUS; на реплике, проверка
![скрин](https://github.com/dmitriypronin48/relica-mysql/blob/main/img/z1-5.jpg)


