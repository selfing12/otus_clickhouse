# Домашнее задание 4

## Цель:
Цель этого домашнего задания - помочь вам понять и применить агрегатные функции, функции, работающие с типами данных, и функции, определяемые пользователем (UDF) в ClickHouse.

## Задачи:
1. Используйте агрегатные функции для обобщения данных.
2. Применять функции, работающие с различными типами данных.
3. Создавать и использовать функции, определяемые пользователем (UDF), в ClickHouse.

### Создание таблицы `transactions` и наполнение ее данными.
```sql
CREATE TABLE transactions (
transaction_id UInt32,
user_id UInt32,
product_id UInt32,
quantity UInt8,
price Float32,
transaction_date Date
)
ENGINE = MergeTree()
ORDER BY (transaction_id);

INSERT INTO transactions VALUES
(1, 101, 501, 2, 49.99, '2025-10-20'),
(2, 102, 502, 1, 120.00, '2025-10-21'),
(3, 101, 503, 3, 15.50, '2025-10-22'),
(4, 103, 504, 5, 9.99,  '2025-10-23'),
(5, 104, 505, 1, 299.99, '2025-10-24');
```
## Агрегатные функции.

### Рассчитываем общий доход от всех операций.
```sql
SELECT sum(quantity * price) AS total_revenue
FROM transactions;
```
Вывод:
```sql
SELECT sum(quantity * price) AS total_revenue
FROM transactions

Query id: 563e3477-50bc-4fbd-bd37-0e0690ed4df6

   ┌─────total_revenue─┐
1. │ 616.4199924468994 │
   └───────────────────┘

1 row in set. Elapsed: 0.004 sec.
```
### Средний доход с одной сделки.
```sql
SELECT avg(quantity * price) AS avg_transaction_revenue
FROM transactions;
```
Вывод:
```sql
SELECT avg(quantity * price) AS avg_transaction_revenue
FROM transactions

Query id: 5192c840-ac1c-4bb5-b64c-a812090e7f9d

   ┌─avg_transaction_revenue─┐
1. │      123.28399848937988 │
   └─────────────────────────┘

1 row in set. Elapsed: 0.005 sec.
```
### Общее количество проданной продукции.
```sql
SELECT sum(quantity) AS total_quantity_sold
FROM transactions;
```
Вывод:
```sql
SELECT sum(quantity) AS total_quantity_sold
FROM transactions

Query id: b32e1ab3-ad70-465c-a277-9b617565ca96

   ┌─total_quantity_sold─┐
1. │                  12 │
   └─────────────────────┘

1 row in set. Elapsed: 0.004 sec.
```
### Количество уникальных пользователей, совершивших покупку.
```sql
SELECT countDistinct(user_id) AS unique_users
FROM transactions;
```
Вывод:
```sql
SELECT countDistinct(user_id) AS unique_users
FROM transactions

Query id: 352262f7-cc69-4c9c-8312-5322cd6a322d

   ┌─unique_users─┐
1. │            4 │
   └──────────────┘

1 row in set. Elapsed: 0.005 sec.
```
## Функции для работы с типами данных.

### `transaction_date` в строку формата `YYYY-MM-DD`.
```sql
SELECT toString(transaction_date) AS date_as_string
FROM transactions;
```
Вывод:
```sql
SELECT toString(transaction_date) AS date_as_string
FROM transactions

Query id: c548812b-b460-46bd-b34f-0ce1f17801ca

   ┌─date_as_string─┐
1. │ 2025-10-20     │
2. │ 2025-10-21     │
3. │ 2025-10-22     │
4. │ 2025-10-23     │
5. │ 2025-10-24     │
   └────────────────┘

5 rows in set. Elapsed: 0.002 sec.
```
### Извлечь год и месяц из `transaction_date`.
```sql
SELECT
toYear(transaction_date) AS year,
toMonth(transaction_date) AS month
FROM transactions;
```
Вывод:
```sql
SELECT
    toYear(transaction_date) AS year,
    toMonth(transaction_date) AS month
FROM transactions

Query id: 12a7007e-b4f9-4f02-b710-1c5d7e13150c

   ┌─year─┬─month─┐
1. │ 2025 │    10 │
2. │ 2025 │    10 │
3. │ 2025 │    10 │
4. │ 2025 │    10 │
5. │ 2025 │    10 │
   └──────┴───────┘

5 rows in set. Elapsed: 0.003 sec.
```
### Округлить `price` до ближайшего целого числа.
```sql
SELECT price, round(price) AS price_rounded
FROM transactions;
```
Вывод:
```sql
SELECT
    price,
    round(price) AS price_rounded
FROM transactions

Query id: 6486e024-d6e0-46ba-b6ee-e92b2023171b

   ┌──price─┬─price_rounded─┐
1. │  49.99 │            50 │
2. │    120 │           120 │
3. │   15.5 │            16 │
4. │   9.99 │            10 │
5. │ 299.99 │           300 │
   └────────┴───────────────┘

5 rows in set. Elapsed: 0.003 sec.
```
### Преобразовать `transaction_id` в строку.
```sql
SELECT transaction_id, toString(transaction_id) AS transaction_id_str
FROM transactions;
```
Вывод:
```sql
SELECT
    transaction_id,
    toString(transaction_id) AS transaction_id_str
FROM transactions

Query id: 1a5a57bb-fb62-4e78-9ddb-75d2ba1bb00f

   ┌─transaction_id─┬─transaction_id_str─┐
1. │              1 │ 1                  │
2. │              2 │ 2                  │
3. │              3 │ 3                  │
4. │              4 │ 4                  │
5. │              5 │ 5                  │
   └────────────────┴────────────────────┘

5 rows in set. Elapsed: 0.003 sec.
```
## User-Defined Functions (UDFs).

### Простая UDF для расчета общей стоимости транзакции.

1. Создаем файл `/var/lib/clickhouse/user_scripts/total_price.py`
```bash
[root@clickhouse-sandbox-single-115929 user_scripts]# cat total_price.py
def total_price(quantity, price):
    return quantity * price
```
2. Регистрация в ClickHouse.
```sql
CREATE FUNCTION total_price AS (quantity, price) -> quantity * price;
```
Вывод:
```sql
CREATE FUNCTION total_price AS (quantity, price) -> (quantity * price)

Query id: 2f3aedc8-3012-401c-858d-dede26ab0a8d

Ok.

0 rows in set. Elapsed: 0.004 sec.
```
3. UDF для расчета общей цены для каждой транзакции.
Вывод:
```sql
SELECT
    transaction_id,
    total_price(quantity, price) AS transaction_total
FROM transactions

Query id: 01ff3c6c-bfab-40c1-8c63-847d048eaa83

   ┌─transaction_id─┬─transaction_total─┐
1. │              1 │  99.9800033569336 │
2. │              2 │               120 │
3. │              3 │              46.5 │
4. │              4 │ 49.94999885559082 │
5. │              5 │  299.989990234375 │
   └────────────────┴───────────────────┘

5 rows in set. Elapsed: 0.002 sec.
```
4. UDF для классификации транзакций на «высокоценные» и «малоценные» на основе порогового значения (например, 100).
```sql
CREATE FUNCTION classify_transaction AS (quantity, price) -> if((quantity * price) > 100, 'high_value', 'low_value')
```
5. UDF для категоризации каждой транзакции.
```sql
SELECT
    transaction_id,
    quantity * price AS total,
    classify_transaction(quantity, price) AS category
FROM transactions;
```
Вывод:
```sql
SELECT
    transaction_id,
    quantity * price AS total,
    classify_transaction(quantity, price) AS category
FROM transactions

Query id: 1004e9d0-573e-41d9-85a9-73e6b608b647

   ┌─transaction_id─┬─────────────total─┬─category───┐
1. │              1 │  99.9800033569336 │ low_value  │
2. │              2 │               120 │ high_value │
3. │              3 │              46.5 │ low_value  │
4. │              4 │ 49.94999885559082 │ low_value  │
5. │              5 │  299.989990234375 │ high_value │
   └────────────────┴───────────────────┴────────────┘

5 rows in set. Elapsed: 0.003 sec.
```