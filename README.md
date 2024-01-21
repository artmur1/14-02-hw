# Домашнее задание к занятию 2 "`SQL`" - `Мурчин Артем`

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

### Решение 1

    version: "3.9"
    services:
      postgres:
        image: postgres:12-alpine3.18
        container_name: postgres
        restart: unless-stopped
        environment:
          POSTGRES_USER: "user"
          POSTGRES_PASSWORD: "secret"
          POSTGRES_ROOT_PASSWORD: "rootsecret"
          PGDATA: "/var/lib/postgresql/data/pgdata"
        volumes:
          - ../2. Init Database:/docker-entrypoint-initdb.d
          - db-data:/var/lib/postgresql/data
          - backup:/var/lib/postgresql/data
        ports:
          - "5432:5432"
    
    volumes:
      db-data:
      backup:

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

### Решение 2

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-02-1.png)

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-02-2.png)

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-02-3.png)

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-02-4.png)

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

### Решение 3

Наполнение таблиц данными:

    INSERT INTO orders (name, price) VALUES ('Chocolate', '10'), ('Printer', '3000'), ('Book', '500'), ('Monitor', '7000'), ('Guitar', '4000');
    INSERT INTO clients (surname, country) VALUES ('Иванов Иван Иванович', 'USA'), ('Петров Петр Петрович', 'Canada'), ('Иоганн Себастьян Бах', 'Japan'), ('Ронни Джеймс Дио', 'Russia'), ('Ritchie Blackmore', 'Russia');

Вычисление количества записей для каждой таблицы:

    select count(*) as "count clients" from clients c ;
    select count(*) as "count orders"  from orders o ;

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-03-1.png)

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-03-2.png)

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

### Решение 4

    UPDATE clients SET orders = 3 where id  = 1;
    UPDATE clients SET orders = 4 where id  = 2;
    UPDATE clients SET orders = 5 where id  = 3;
    select surname, orders from clients c
    where orders > 0;

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-04-2.png)

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

### Решение 5

В результате выполнения запроса Explain видно какая стоимость запроса "cost", количество обрабатываемых строк "rows".

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-05-1.png)

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

### Решение 6

    pg_dump -d test_db -U user -h 192.168.1.124 -p 5432 > /tmp/test_db.dump

    create database test_db;

    psql -d test_db -U user -h 192.168.1.124 -p 15432 < /tmp/test_db.dump

![alt text](https://github.com/artmur1/14-02-hw/blob/main/14-02-hw-06-1.png)

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

