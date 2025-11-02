# Домашнее задание 6 Словари, оконные и табличные функции.

## Цель:
Закрепить навыки работы со словарями и оконными функциями для выборки данных и вычисления аккумулятивной суммы.

## Задачи:

1. Создать таблицу с полями.
2. Создать словарь, где ключ `user_id`, атрибут email `(String)`, источник словаря выбрать любой удобный, например, файл.
3. SELECT, который возвращает:
  - email с помощью dictGet,
    аккумулятивную сумму expense с окном по action,
    сортировку по email.

### Создание таблицы.
```sql
CREATE TABLE IF NOT EXISTS user_actions
(
    `user_id` UInt64,
    `action` String,
    `expense` UInt64
)
ENGINE = MergeTree
ORDER BY (user_id, action)

Query id: 80cf933b-1c67-4f0a-b4bc-6cf5eca511ca

Ok.

0 rows in set. Elapsed: 0.008 sec.
```
### Подготовка источника для словаря.

```bash
[root@clickhouse-sandbox-single-115929 user_scripts]# cat /var/lib/clickhouse/user_files/user_emails.csv
user_id,email
1,freedpary@gmail.com
2,saint12@bk.ru
3,selfing@list.ru
4,sobakabalabaka@mail.ru
5,dota2frends@yaho.com
```
### Создайте словарь.
```sql
CREATE DICTIONARY IF NOT EXISTS user_email_dict
(
    `user_id` UInt64,
    `email` String
)
PRIMARY KEY user_id
SOURCE(FILE(PATH '/var/lib/clickhouse/user_files/user_emails.csv' FORMAT 'CSV'))
LIFETIME(MIN 300 MAX 600)
LAYOUT(FLAT())

Query id: a2a4f8f7-9054-4e94-8541-962285fa3c15

Ok.

0 rows in set. Elapsed: 0.019 sec.
```
### Наполняем таблицу и источник данными с низкоардинальными значениями для поля action и несколькими повторяющимися строками для каждого user_id.
```sql
clickhouse-server :) INSERT INTO user_actions VALUES
(1, 'login', 100),
(1, 'purchase', 200),
(1, 'logout', 50),
(2, 'login', 150),
(2, 'purchase', 300),
(2, 'logout', 80),
(3, 'login', 120),
(3, 'purchase', 180),
(3, 'logout', 80);
(4, 'login', 120),
(4, 'purchase', 130),
(4, 'logout', 40),
(5, 'login', 160),
(5, 'purchase', 190),
(5, 'logout', 10);
```
### SELECT, который возвращает email с помощью dictGet.
```sql
SELECT
    dictGetString('user_email_dict', 'email', user_id) AS email,
    action,
    expense,
    sum(expense) OVER (PARTITION BY action ORDER BY user_id ASC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_expense
FROM user_actions
ORDER BY email ASC

Query id: de5027ec-3ec1-45aa-81ce-bf3758e8f663

    ┌─email──────────────────┬─action───┬─expense─┬─cumulative_expense─┐
 1. │ dota2frends@yaho.com   │ login    │     160 │               1020 │
 2. │ dota2frends@yaho.com   │ logout   │      10 │                470 │
 3. │ dota2frends@yaho.com   │ purchase │     190 │               1680 │
 4. │ freedpary@gmail.com    │ login    │     100 │                100 │
 5. │ freedpary@gmail.com    │ login    │     100 │                200 │
 6. │ freedpary@gmail.com    │ logout   │      50 │                 50 │
 7. │ freedpary@gmail.com    │ logout   │      50 │                100 │
 8. │ freedpary@gmail.com    │ purchase │     200 │                200 │
 9. │ freedpary@gmail.com    │ purchase │     200 │                400 │
10. │ saint12@bk.ru          │ login    │     150 │                350 │
11. │ saint12@bk.ru          │ login    │     150 │                500 │
12. │ saint12@bk.ru          │ logout   │      80 │                180 │
13. │ saint12@bk.ru          │ logout   │      80 │                260 │
14. │ saint12@bk.ru          │ purchase │     300 │                700 │
15. │ saint12@bk.ru          │ purchase │     300 │               1000 │
16. │ selfing@list.ru        │ login    │     120 │                620 │
17. │ selfing@list.ru        │ login    │     120 │                740 │
18. │ selfing@list.ru        │ logout   │      80 │                340 │
19. │ selfing@list.ru        │ logout   │      80 │                420 │
20. │ selfing@list.ru        │ purchase │     180 │               1180 │
21. │ selfing@list.ru        │ purchase │     180 │               1360 │
22. │ sobakabalabaka@mail.ru │ login    │     120 │                860 │
23. │ sobakabalabaka@mail.ru │ logout   │      40 │                460 │
24. │ sobakabalabaka@mail.ru │ purchase │     130 │               1490 │
    └────────────────────────┴──────────┴─────────┴────────────────────┘

24 rows in set. Elapsed: 0.005 sec.
```
