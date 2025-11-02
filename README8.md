# Домашнее задание 8 Движки MergeTree.

## Цель:
Изучить принципы работы движка MergeTree в ClickHouse, рассмотреть выбор движка для различных задач, научиться дедуплицировать данные и заменять операции delete/update.

## Задачи:
1. По заданным описаниям таблиц и вставки данных определить используемый движок.
2. Заполнить пропуски, запустить код.
3. Сравнить полученный вывод и результат из условия.

### Создание таблицы `tbl1`.
```sql

CREATE TABLE tbl1
(
`UserID` UInt64,
`PageViews` UInt8,
`Duration` UInt8,
`Sign` Int8,
`Version` UInt8
)
ENGINE = ReplacingMergeTree(Sign)
ORDER BY UserID

Query id: a23d54fb-934a-4088-b27e-cee5c29e9a34

Ok.

0 rows in set. Elapsed: 0.007 sec.
```

### Вставка Данных.
```sql
clickhouse-server :) INSERT INTO tbl1 VALUES (4324182021466249494, 5, 146, -1, 1);

INSERT INTO tbl1 FORMAT Values

Query id: 2a4b4122-a880-42d6-a060-2bd606f14b08

Ok.

1 row in set. Elapsed: 0.006 sec.
```
```sql
clickhouse-server :) INSERT INTO tbl1 VALUES (4324182021466249494, 5, 146, 1, 1),(4324182021466249494, 6, 185, 1, 2);

INSERT INTO tbl1 FORMAT Values

Query id: cb3b19ae-1828-4607-ae20-69569c8f6bbe

Ok.

2 rows in set. Elapsed: 0.005 sec.
```
### Проверка данных.

```sql
clickhouse-server :) SELECT * FROM tbl1;

SELECT *
FROM tbl1

Query id: 16a8f1c9-ae75-49eb-ae30-2a8f16dc37c1

┌──────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
1. │ 4324182021466249494 │         6 │      185 │    1 │       2 │
2. │ 4324182021466249494 │         5 │      146 │   -1 │       1 │
   └─────────────────────┴───────────┴──────────┴──────┴─────────┘

2 rows in set. Elapsed: 0.003 sec.
```
```sql
clickhouse-server :) SELECT * FROM tbl1 FINAL;

SELECT *
FROM tbl1
FINAL

Query id: 0549507f-3c50-4b3c-866e-15b997b32b93

┌──────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
1. │ 4324182021466249494 │         6 │      185 │    1 │       2 │
   └─────────────────────┴───────────┴──────────┴──────┴─────────┘

1 row in set. Elapsed: 0.003 sec.
```

### Создание таблицы `tbl2`.
```sql
CREATE TABLE tbl2
(
    `key` UInt32,
    `value` UInt32
)
ENGINE = SummingMergeTree
ORDER BY key

Query id: cfefebb5-a69c-44cc-a485-a4242dcaf5ad

Ok.

0 rows in set. Elapsed: 0.009 sec.
```

### Вставка Данных.
```sql
clickhouse-server :) INSERT INTO tbl2 VALUES (1,1),(1,2),(2,1);

INSERT INTO tbl2 FORMAT Values

Query id: 4cbaaf00-2962-479f-8ddf-80b09ec93674

Ok.

3 rows in set. Elapsed: 0.005 sec.
```
### Проверка данных.

```sql
clickhouse-server :) SELECT * FROM tbl2;

SELECT *
FROM tbl2

Query id: 50568fc6-d67b-4fe1-a819-9afb4c32c108

   ┌─key─┬─value─┐
1. │   1 │     3 │
2. │   2 │     1 │
   └─────┴───────┘

2 rows in set. Elapsed: 0.002 sec.
```

### Создание таблицы `tbl3`.
```sql
CREATE TABLE tbl3
(
    `id` Int32,
    `status` String,
    `price` String,
    `comment` String
)
ENGINE = ReplacingMergeTree()
PRIMARY KEY (id)
ORDER BY (id, status);
```

### Вставка Данных.
```sql
clickhouse-server :) INSERT INTO tbl3 VALUES (23, 'success', '1000', 'Confirmed');

INSERT INTO tbl3 FORMAT Values

Query id: f162e8c3-c026-43d0-9abd-fa673bc34b5f

Ok.

1 row in set. Elapsed: 0.005 sec.

clickhouse-server :) INSERT INTO tbl3 VALUES (23, 'success', '2000', 'Cancelled');

INSERT INTO tbl3 FORMAT Values

Query id: c4da3d67-6a6e-43c4-9b42-6e5110c2c5d9

Ok.

1 row in set. Elapsed: 0.006 sec.
```
### Проверка данных.

```sql
clickhouse-server :) SELECT * from tbl3 WHERE id=23;

SELECT *
FROM tbl3
WHERE id = 23

Query id: b6a4a789-dbe6-4d9b-8ff6-6c43bfa0b735

   ┌─id─┬─status──┬─price─┬─comment───┐
1. │ 23 │ success │ 2000  │ Cancelled │
2. │ 23 │ success │ 1000  │ Confirmed │
   └────┴─────────┴───────┴───────────┘

2 rows in set. Elapsed: 0.004 sec.

clickhouse-server :) SELECT * from tbl3 FINAL WHERE id=23;

SELECT *
FROM tbl3
FINAL
WHERE id = 23

Query id: c5155f16-472f-471a-a4cb-e4896daf7f8e

   ┌─id─┬─status──┬─price─┬─comment───┐
1. │ 23 │ success │ 2000  │ Cancelled │
   └────┴─────────┴───────┴───────────┘

1 row in set. Elapsed: 0.004 sec.
```
### Создание таблицы `tbl4`.
```sql
CREATE TABLE tbl4
(
    CounterID UInt8,
    StartDate Date,
    UserID UInt64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate);
```

### Вставка Данных.
```sql
clickhouse-server :) INSERT INTO tbl4 VALUES (0, '2019-11-11', 1);
INSERT INTO tbl4 VALUES (1, '2019-11-12', 1);

INSERT INTO tbl4 FORMAT Values

Query id: 1b3a8a21-3c68-4882-837a-c3b5321ed8b2

Ok.

1 row in set. Elapsed: 0.005 sec.


INSERT INTO tbl4 FORMAT Values

Query id: 543f4668-962a-447b-8b86-48a5cccbcc0b

Ok.

1 row in set. Elapsed: 0.004 sec.
```
### Проверка данных.

```sql
clickhouse-server :) SELECT * FROM tbl4;

SELECT *
FROM tbl4

Query id: a661e165-58dc-473d-a23c-2017ff638861

   ┌─CounterID─┬──StartDate─┬─UserID─┐
1. │         1 │ 2019-11-12 │      1 │
2. │         0 │ 2019-11-11 │      1 │
   └───────────┴────────────┴────────┘

2 rows in set. Elapsed: 0.003 sec.
```
### Создание таблицы `tbl5`.
```sql
CREATE TABLE tbl5
(
    CounterID UInt8,
    StartDate Date,
    UserID AggregateFunction(uniq, UInt64)
)
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate);
```

### Вставка Данных.
```sql
INSERT INTO tbl5 SELECT
    CounterID,
    StartDate,
    uniqState(UserID)
FROM tbl4
GROUP BY
    CounterID,
    StartDate

Query id: 3fb27ca8-a4a3-4a76-b9dc-b472f0e83503

Ok.

0 rows in set. Elapsed: 0.010 sec.
```
```sql
INSERT INTO tbl5 VALUES (1,'2019-11-12',1);

INSERT INTO tbl5 FORMAT Values

Query id: d177320d-1011-4471-a3e0-8152055e1787

Ok.
Error on processing query: Code: 53. DB::Exception: Cannot convert UInt64 to AggregateFunction(uniq, UInt64): While executing ValuesBlockInputFormat: data for INSERT was parsed from query. (TYPE_MISMATCH) (version 25.3.2.39 (official build))
```
### Проверка данных.

```sql
SELECT uniqMerge(UserID) AS state
FROM tbl5
GROUP BY
    CounterID,
    StartDate

Query id: 3c79f760-ede1-4f17-a0ce-bfcaeb29f9b6

   ┌─state─┐
1. │     1 │
2. │     1 │
   └───────┘

2 rows in set. Elapsed: 0.005 sec.
```

### Создание таблицы `tbl6`.
```sql
CREATE TABLE tbl6
(
    id Int32,
    status String,
    price String,
    comment String,
    sign Int8
)
ENGINE = CollapsingMergeTree(sign)
PRIMARY KEY id
ORDER BY (id, status);
```

### Вставка Данных.
```sql
clickhouse-server :) INSERT INTO tbl6 VALUES (23, 'success', '1000', 'Confirmed', 1);
INSERT INTO tbl6 VALUES (23, 'success', '1000', 'Confirmed', -1), (23, 'success', '2000', 'Cancelled', 1);


INSERT INTO tbl6 FORMAT Values

Query id: e86dd20c-07b3-4334-99de-ad7956e14fb0

Ok.

1 row in set. Elapsed: 0.005 sec.


INSERT INTO tbl6 FORMAT Values

Query id: 6af4d95a-a5dc-4ff3-9315-85441ccae767

Ok.

2 rows in set. Elapsed: 0.004 sec.
```
### Проверка данных.

```sql
clickhouse-server :) SELECT * FROM tbl6;

SELECT *
FROM tbl6

Query id: 873e2039-bb83-4259-9b18-53f4494516f2

   ┌─id─┬─status──┬─price─┬─comment───┬─sign─┐
1. │ 23 │ success │ 1000  │ Confirmed │   -1 │
2. │ 23 │ success │ 2000  │ Cancelled │    1 │
3. │ 23 │ success │ 1000  │ Confirmed │    1 │
   └────┴─────────┴───────┴───────────┴──────┘

3 rows in set. Elapsed: 0.004 sec.
```

```sql
clickhouse-server :) SELECT * FROM tbl6 FINAL;

SELECT *
FROM tbl6
FINAL

Query id: 8c4c22a9-70ef-4a1b-b19d-8027948ccac4

   ┌─id─┬─status──┬─price─┬─comment───┬─sign─┐
1. │ 23 │ success │ 2000  │ Cancelled │    1 │
   └────┴─────────┴───────┴───────────┴──────┘

1 row in set. Elapsed: 0.003 sec.
```