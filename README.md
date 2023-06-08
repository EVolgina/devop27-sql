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
---
|Наименование	| цена |
|Шоколад	    |10    |
|Принтер	    |3000  | 
Книга	500
Монитор	7000
Гитара	4000
Таблица clients

ФИО	Страна проживания
Иванов Иван Иванович	USA
Петров Петр Петрович	Canada
Иоганн Себастьян Бах	Japan
Ронни Джеймс Дио	Russia
Ritchie Blackmore	Russia
Используя SQL-синтаксис:

вычислите количество записей для каждой таблицы.
Приведите в ответе:
- запросы,
- результаты их выполнения.
### Ответ
![zd3] (https://github.com/EVolgina/devop27-sql/blob/main/zd3.PNG)
```
INSERT INTO orders (id, name, price)
VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
 
INSERT INTO clients (id, last_name, country_of_residence, order_id)
VALUES (1, 'Иванов Иван Иванович', 'USA', 1), (2, 'Петров Петр Петрович', 'Canada', 2), (3, 'Иоганн Себастьян Бах', 'Japan', 3), (4, 'Ронни Джеймс Дио', 'Russia', 4), (5, 'Ritchie Blackmore', 'Russia', 5);

SELECT COUNT(*) AS order_count FROM orders;
SELECT COUNT(*) AS client_count FROM clients;
```
# Задача 4
Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

ФИО	Заказ
Иванов Иван Иванович	Книга
Петров Петр Петрович	Монитор
Иоганн Себастьян Бах	Гитара
Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
Подсказка: используйте директиву UPDATE.

# Задача 5
Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).
Приведите получившийся результат и объясните, что значат полученные значения.

# Задача 6
Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).
Остановите контейнер с PostgreSQL, но не удаляйте volumes.
Поднимите новый пустой контейнер с PostgreSQL.
Восстановите БД test_db в новом контейнере.
Приведите список операций, который вы применяли для бэкапа данных и восстановления.
