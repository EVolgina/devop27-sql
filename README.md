# Задача 1
Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.
Приведите получившуюся команду или docker-compose-манифест.
### Ответ:
1. Необходимо создать yml файл, который настраивает экземпляр PostgreSQL с двумя томами для данных базы данных и резервных копий
```
version: '3'
services:
  postgres:
    image: postgres:12
    restart: always
    environment: 
      - POSTGRES_USER: admin
      - POSTGRES_PASSWORD: admin
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data
      - backups:/backups
volumes:
  pgdata:
  backups:
```
![ps](https://github.com/EVolgina/devop27-sql/blob/main/pssql.PNG)
![zap](https://github.com/EVolgina/devop27-sql/blob/main/docker%20ps.PNG)
Подключаемся к docker контейнеру и переходим к задаче 2 (sudo docker exec -it 28852d151b64 psql -U admin -d admin)

# Задача 2
В БД из задачи 1:
- создайте пользователя test-admin-user и БД test_db; 
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.\
Таблица orders:
- id (serial primary key);
- наименование (string);
- цена (integer).\
Таблица clients:
- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).\
Приведите:
- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.
![1](https://github.com/EVolgina/devop27-sql/blob/main/%D1%81%D0%BF%D0%B8%D1%81%D0%BE%D0%BA%20%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86.PNG)
![2](https://github.com/EVolgina/devop27-sql/blob/main/rol.PNG)
```
SELECT grantee, table_name, privilege_type
FROM information_schema.role_table_grants
WHERE table_schema = 'public' AND table_catalog = 'test_db';
```
# Задача 3
Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:\
Таблица orders
----------------------
Наименование	| цена 
--------------|--------
Шоколад	    |10    
--------------|--------
Принтер	    |3000   
--------------|--------
Книга     	  |500   
--------------| --------
Монитор	    |7000  
--------------| --------
Гитара	      |4000  
--------------|--------

Таблица clients
--------------------------------
ФИО	        |Страна проживания
----------------------|---------
Иванов Иван Иванович	|USA
----------------------|---------
Петров Петр Петрович	|Canada
----------------------|---------
Иоганн Себастьян Бах	|Japan
----------------------|---------
Ронни Джеймс Дио	|Russia
----------------------|---------
Ritchie Blackmore	|Russia
-----------------------|---------

Используя SQL-синтаксис:
вычислите количество записей для каждой таблицы.
Приведите в ответе:
- запросы,
- результаты их выполнения.
### Ответ
```
INSERT INTO orders (id, name, price)
VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
 
INSERT INTO clients (id, last_name, country_of_residence, order_id)
VALUES (1, 'Иванов Иван Иванович', 'USA', 1), (2, 'Петров Петр Петрович', 'Canada', 2), (3, 'Иоганн Себастьян Бах', 'Japan', 3), (4, 'Ронни Джеймс Дио', 'Russia', 4), (5, 'Ritchie Blackmore', 'Russia', 5);

SELECT COUNT(*) AS order_count FROM orders;
SELECT COUNT(*) AS client_count FROM clients;
```
![zd3](https://github.com/EVolgina/devop27-sql/blob/main/zd3.PNG)

# Задача 4
Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.
Используя foreign keys, свяжите записи из таблиц, согласно таблице:

------------------------
ФИО	             |Заказ
-----------------|------
Иванов Иван Иванович |	Книга
-------------------- |-------
Петров Петр Петрович |	Монитор
-------------------- |--------
Иоганн Себастьян Бах |	Гитара
-------------------- |--------

Приведите SQL-запросы для выполнения этих операций.
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
Подсказка: используйте директиву UPDATE.

### Ответ:
![zd4](https://github.com/EVolgina/devop27-sql/blob/main/sz4.PNG)
```
UPDATE clients
SET order_id = (SELECT id FROM orders WHERE name = 'Книга')
WHERE last_name = 'Иванов Иван Иванович' AND country_of_residence = 'USA';
UPDATE clients
SET order_id = (SELECT id FROM orders WHERE name = 'Монитор')
WHERE last_name = 'Петров Петр Петрович' AND country_of_residence = 'Canada';
UPDATE clients
SET order_id = (SELECT id FROM orders WHERE name = 'Гитара')
WHERE last_name = 'Иоганн Себастьян Бах’ AND country_of_residence = 'Japan';

SELECT c.last_name, o.name AS order_name
FROM clients c
JOIN orders o ON c.order_id = o.id;

```

# Задача 5
Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).
Приведите получившийся результат и объясните, что значат полученные значения.
### Ответ:
Команда EXPLAIN предоставляет информацию о том, как выполняется запрос, и помогает анализировать производительность запроса.\
Команда EXPLAIN предоставляет информацию о том, как планировщик базы данных намеревается выполнить запрос.\
Выходные данные EXPLAIN показывают, что запрос выполнит хэш-соединение между таблицами clients и orders. По его оценкам, запрос вернет\ приблизительно 810 строк шириной 64. Стоимость представляет собой оценку времени выполнения запроса.\

```
EXPLAIN SELECT c.last_name, o.name AS order_name
FROM clients c
JOIN orders o ON c.order_id = o.id;

```
![zd5](https://github.com/EVolgina/devop27-sql/blob/main/zd5.PNG)

# Задача 6
Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).
Остановите контейнер с PostgreSQL, но не удаляйте volumes.
Поднимите новый пустой контейнер с PostgreSQL.
Восстановите БД test_db в новом контейнере.
Приведите список операций, который вы применяли для бэкапа данных и восстановления.
### Ответ:
```
vagrant@server1:~/sql$  sudo docker exec -it 28852d151b64 psql -U admin -d admin
psql (12.15 (Debian 12.15-1.pgdg110+1))
Type "help" for help.

admin=# pg_dump -U admin  test_db > /backup/test_db.backup
admin-# \q
vagrant@server1:~/sql$ sudo docker stop 28852d151b64
28852d151b64
vagrant@server1:~/sql$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS                          PORTS     NAMES
ac264b651e84   prom/prometheus:latest   "/bin/prometheus --c…"   11 days ago   Restarting (2) 11 seconds ago             prometheus1
vagrant@server1:~/sql$ mkdir new
vagrant@server1:~/sql$ cd new
vagrant@server1:~/sql/new$ sudo nano docker-compose.yml
vagrant@server1:~/sql/new$ sudo docker run --name pg12 -e POSTGRES_USER=admin1 -e POSTGRES_PASSWORD=admin1 -d postgres:12
7147e775dc6498b67225c5b6594654281784e47c68394e797b4cd3060fe4a669
vagrant@server1:~/sql$ sudo docker cp 28852d151b64:/backups/backup_file.dump /home/vagrant/sql/backup_file.dump
Successfully copied 7.68kB to /home/vagrant/sql/backup_file.dump
vagrant@server1:~/sql$ sudo docker cp /home/vagrant/sql/backup_file.dump 7147e775dc64:/home/backup_file.dump
Successfully copied 7.68kB to 7147e775dc64:/home/backup_file.dump
vagrant@server1:~/sql$ sudo docker exec -it 7147e775dc64 psql -U admin1 -d admin1
psql (12.15 (Debian 12.15-1.pgdg110+1))
Type "help" for help.
admin1=# CREATE DATABASE test_db;
CREATE DATABASE
admin1=# GRANT ALL PRIVILEGES ON DATABASE "test_db" to admin1;
GRANT
admin1=# CREATE USER admin WITH PASSWORD 'admin';
CREATE ROLE
admin1=# GRANT ALL PRIVILEGES ON DATABASE "test_db" to admin;
GRANT
admin1=#\q
vagrant@server1:~/sql$ sudo docker exec -it 7147e775dc64 pg_restore -U admin -d test_db /home/backup_file.dump
```

