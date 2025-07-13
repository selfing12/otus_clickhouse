# Домашнее задание 3
## Работа с SQL в ClickHouse

## Цели:
- Отработать основные sql-операции.
- Изучить на практике особенности sql-диалекта.

## Выполнение задания

1. Создаем нашу таблицу с меню:
```sql
CREATE TABLE menu_values
(
id UInt32 COMMENT 'ID блюда',
name LowCardinality(String) COMMENT 'Название блюда',
description Nullable(String) COMMENT 'Описание блюда',
price Float32 COMMENT 'Цена блюда в рублях',
available UInt8 COMMENT 'Наличие блюда (1 - есть, 0 - нет)'
) ENGINE = MergeTree()
ORDER BY id;
```
2. Делаем вставку из 5 столбцов:

```sql
INSERT INTO menu_values VALUES
(1, 'Solyanka', 'Russian beet soup', 5.99, 1),
(2, 'Pelmeni', NULL, 7.50, 1),
(3, 'Blini', 'Thin pancakes', 4.00, 0),
(4, 'Okroshka', 'Traditional summer soup', 6.25, 1),
(5, 'Kombucha', 'Fermented drink', 2.00, 1);

```

3. Проверяем колличество столбцов:
```sql
select count () from menu_values;

SELECT count()
FROM menu_values

Query id: 22530626-bb74-43b3-b26f-2a05694df624

   ┌─count()─┐
1. │       5 │
   └─────────┘

1 row in set. Elapsed: 0.003 sec.
```
4. Проверяем их заполнение:
```sql
clickhouse-server :) select * from menu_values;

SELECT *
FROM menu_values

Query id: 169612bb-391b-4060-8d5c-31b115d28c7a

   ┌─id─┬─name─────┬─description─────────────┬─price─┬─available─┐
1. │  1 │ Solyanka │ Russian beet soup       │  5.99 │         1 │
2. │  2 │ Pelmeni  │ ᴺᵁᴸᴸ                    │   7.5 │         1 │
3. │  3 │ Blini    │ Thin pancakes           │     4 │         0 │
4. │  4 │ Okroshka │ Traditional summer soup │  6.25 │         1 │
5. │  5 │ Kombucha │ Fermented drink         │     2 │         1 │
   └────┴──────────┴─────────────────────────┴───────┴───────────┘

5 rows in set. Elapsed: 0.003 sec.

```

## CRUD операции:

Вывод пунктов меню в состоянии `available`
```sql
SELECT * FROM menu_values WHERE available = 1;

SELECT *
FROM menu_values
WHERE available = 1

Query id: b1d27357-a6a3-4449-bb5e-14ac476597a7

   ┌─id─┬─name─────┬─description─────────────┬─price─┬─available─┐
1. │  1 │ Solyanka │ Russian beet soup       │  5.99 │         1 │
2. │  2 │ Pelmeni  │ ᴺᵁᴸᴸ                    │   7.5 │         1 │
3. │  4 │ Okroshka │ Traditional summer soup │  6.25 │         1 │
4. │  5 │ Kombucha │ Fermented drink         │     2 │         1 │
   └────┴──────────┴─────────────────────────┴───────┴───────────┘

4 rows in set. Elapsed: 0.004 sec.
```

Изеняем цену у третьего пункта меню:
```sql
ALTER TABLE menu_values UPDATE price = 6.50 WHERE id = 3;

ALTER TABLE menu_values
    (UPDATE price = 6.5 WHERE id = 3)

Query id: ee8b31b4-36f1-429e-ad5c-27be75d2d86c

Ok.

0 rows in set. Elapsed: 0.004 sec.
```

Проверяем:
```sql
select * from menu_values WHERE id = 3;

SELECT *
FROM menu_values
WHERE id = 3

Query id: a9352ca8-8499-477d-894e-f7d2d292c64a

   ┌─id─┬─name──┬─description───┬─price─┬─available─┐
1. │  3 │ Blini │ Thin pancakes │   6.5 │         0 │
   └────┴───────┴───────────────┴───────┴───────────┘

1 row in set. Elapsed: 0.003 sec.

```

Удаляем один пункт например 5
```sql
alter table menu_values delete where id = 5;

alter table menu_values
    (elete where id = 5)

Query id: 70872ee4-f347-4a2c-afe7-e9e615fdbdeb

Ok.

0 rows in set. Elapsed: 0.004 sec.
```

Проверяем:
```sql
select * from menu_values;

SELECT *
FROM menu_values

Query id: 937c9270-850c-445f-8746-d1b67c501179

   ┌─id─┬─name─────┬─description─────────────┬─price─┬─available─┐
1. │  1 │ Solyanka │ Russian beet soup       │  5.99 │         1 │
2. │  2 │ Pelmeni  │ ᴺᵁᴸᴸ                    │   7.5 │         1 │
3. │  3 │ Blini    │ Thin pancakes           │   6.5 │         0 │
4. │  4 │ Okroshka │ Traditional summer soup │  6.25 │         1 │
   └────┴──────────┴─────────────────────────┴───────┴───────────┘

4 rows in set. Elapsed: 0.003 sec.
```

Добавляем колонку Категорий блюд
```sql
alter table menu_values add column category LowCardinality(String) default 'Main' comment 'Категория блюда';

ALTER TABLE menu_values
    (ADD COLUMN `category` LowCardinality(String) DEFAULT 'Main' COMMENT 'Категория блюда')

Query id: 040e7e94-ac71-4306-b92d-8f3862c3ebee

Ok.

0 rows in set. Elapsed: 0.005 sec.

```
Проверяем:
```sql
select * from menu_values;

SELECT *
FROM menu_values

Query id: 85081690-ac6b-415f-b212-fea3ca7d0f3c

   ┌─id─┬─name─────┬─description─────────────┬─price─┬─available─┬─category─┐
1. │  1 │ Solyanka │ Russian beet soup       │  5.99 │         1 │ Main     │
2. │  2 │ Pelmeni  │ ᴺᵁᴸᴸ                    │   7.5 │         1 │ Main     │
3. │  3 │ Blini    │ Thin pancakes           │   6.5 │         0 │ Main     │
4. │  4 │ Okroshka │ Traditional summer soup │  6.25 │         1 │ Main     │
   └────┴──────────┴─────────────────────────┴───────┴───────────┴──────────┘

4 rows in set. Elapsed: 0.003 sec.
```

Добавляем колонку Калорийности:
```sql
alter table menu_values add column calories UInt16 default 0 comment 'Калории блюда';

ALTER TABLE menu_values
(ADD COLUMN `calories` UInt16 DEFAULT 0 COMMENT 'Калории блюда')

Query id: d4c3bb0e-1f06-40e4-8884-c7e7c0a4c819

Ok.

0 rows in set. Elapsed: 0.007 sec.
```
Проверяем:
```sql
select * from menu_values;

SELECT *
FROM menu_values

Query id: 4fe46517-a8b6-4f66-b503-6ca752746c37

   ┌─id─┬─name─────┬─description─────────────┬─price─┬─available─┬─category─┬─calories─┐
1. │  1 │ Solyanka │ Russian beet soup       │  5.99 │         1 │ Main     │        0 │
2. │  2 │ Pelmeni  │ ᴺᵁᴸᴸ                    │   7.5 │         1 │ Main     │        0 │
3. │  3 │ Blini    │ Thin pancakes           │   6.5 │         0 │ Main     │        0 │
4. │  4 │ Okroshka │ Traditional summer soup │  6.25 │         1 │ Main     │        0 │
   └────┴──────────┴─────────────────────────┴───────┴───────────┴──────────┴──────────┘

4 rows in set. Elapsed: 0.003 sec.
```

Удаляем колонки Описания и Доступности блюд:
```sql
alter table menu_values drop column available;

ALTER TABLE menu_values
(DROP COLUMN available)

Query id: 38c35ef5-f540-4948-8f31-a01d0f9e81f1

Ok.

0 rows in set. Elapsed: 0.009 sec.
```
```sql
alter table menu_values drop column description;

ALTER TABLE menu_values
(DROP COLUMN description)

Query id: 3fbb8caf-0c69-4dba-bce1-54b24e3a2f19

Ok.

0 rows in set. Elapsed: 0.010 sec.
```
Проверяем:
```sql
select * from menu_values;

SELECT *
FROM menu_values

Query id: ba5bc91a-92e3-4143-844d-efc36922372d

   ┌─id─┬─name─────┬─price─┬─category─┬─calories─┐
1. │  1 │ Solyanka │  5.99 │ Main     │        0 │
2. │  2 │ Pelmeni  │   7.5 │ Main     │        0 │
3. │  3 │ Blini    │   6.5 │ Main     │        0 │
4. │  4 │ Okroshka │  6.25 │ Main     │        0 │
   └────┴──────────┴───────┴──────────┴──────────┘

4 rows in set. Elapsed: 0.003 sec.
```
### Материализация путем создания копии таблицы:

1. Создаем материализованное представление
```sql
CREATE TABLE menu_values_copy AS restaurant_menu.menu_values ENGINE = MergeTree() ORDER BY id;

CREATE TABLE menu_values_copy AS restaurant_menu.menu_values
ENGINE = MergeTree
ORDER BY id

Query id: 3afb3625-e694-455d-95eb-724432f530e9

Ok.

0 rows in set. Elapsed: 0.005 sec.
```

2. Делаем вставку в него путем использования `select` запроса

```sql
INSERT INTO menu_values_copy SELECT * FROM menu_values;

INSERT INTO menu_values_copy SELECT *
FROM menu_values

Query id: 1d06a0f9-bd03-48e3-93d2-7648a2a9d0cf

Ok.

0 rows in set. Elapsed: 0.005 sec.
```
### Выборка данных (select) из таблицы `system.numbers`
```sql
SELECT * FROM system.numbers LIMIT 10;

SELECT *
FROM system.numbers
LIMIT 10

Query id: 7d783866-055e-4ad7-aaac-70bf7fbd3179

    ┌─number─┐
1.  │      0 │
2.  │      1 │
3.  │      2 │
4.  │      3 │
5.  │      4 │
6.  │      5 │
7.  │      6 │
8.  │      7 │
9.  │      8 │
10. │      9 │
    └────────┘
```
### Выборка данных (select) из таблицы `system.tables`
```sql
SELECT * FROM system.tables LIMIT 1;

SELECT *
FROM system.tables
LIMIT 1

Query id: 810b8105-624d-4e3b-872b-0686f7ab54fc

Row 1:
──────
database:                         INFORMATION_SCHEMA
name:                             CHARACTER_SETS
uuid:                             00000000-0000-0000-0000-000000000000
engine:                           View
is_temporary:                     0
data_paths:                       []
metadata_path:
metadata_modification_time:       1970-01-01 03:00:00
metadata_version:                 0
dependencies_database:            []
dependencies_table:               []
create_table_query:               CREATE VIEW INFORMATION_SCHEMA.CHARACTER_SETS (`character_set_name` String, `CHARACTER_SET_NAME` String) SQL SECURITY INVOKER AS SELECT arrayJoin(['utf8', 'utf8mb4', 'ascii', 'binary']) AS character_set_name, character_set_name AS CHARACTER_SET_NAME
engine_full:
as_select:                        SELECT arrayJoin(['utf8', 'utf8mb4', 'ascii', 'binary']) AS character_set_name, character_set_name AS CHARACTER_SET_NAME
partition_key:
sorting_key:
primary_key:
sampling_key:
storage_policy:
total_rows:                       ᴺᵁᴸᴸ
total_bytes:                      ᴺᵁᴸᴸ
total_bytes_uncompressed:         ᴺᵁᴸᴸ
parts:                            ᴺᵁᴸᴸ
active_parts:                     ᴺᵁᴸᴸ
total_marks:                      ᴺᵁᴸᴸ
active_on_fly_data_mutations:     0
active_on_fly_metadata_mutations: 0
lifetime_rows:                    ᴺᵁᴸᴸ
lifetime_bytes:                   ᴺᵁᴸᴸ
comment:
has_own_data:                     0
loading_dependencies_database:    []
loading_dependencies_table:       []
loading_dependent_database:       []
loading_dependent_table:          []

1 row in set. Elapsed: 0.004 sec.
```

### Практика с партициями:

1. Если таблица без партиций то сделаем её с партицией, например по категориям:
```sql
CREATE TABLE menu_values_partitioned
(
    id UInt8,
    name String,
    price Float32,
    category LowCardinality(String),
    calories UInt16
)
ENGINE = MergeTree()
PARTITION BY category
ORDER BY id;
```

2. Делаем `select` запрос для проверки партиций:

```sql
   SELECT
   partition,
   name,
   active
   FROM system.parts
   WHERE table = 'menu_values_partitioned'
   AND database = currentDatabase();
```
Вывод:
```sql
SELECT
    partition,
    name,
    active
FROM system.parts
WHERE (`table` = 'menu_values_partitioned') AND (database = currentDatabase())

Query id: e8874101-2d98-4b21-a9ba-7756ee259100

   ┌─partition─┬─name───────────────────────────────────┬─active─┐
1. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_1_1_0 │      1 │
   └───────────┴────────────────────────────────────────┴────────┘

1 row in set. Elapsed: 0.003 sec.
```

3. Делаем detach партиции Main:
```sql
ALTER TABLE menu_values_partitioned DETACH PARTITION 'Main';

ALTER TABLE menu_values_partitioned
    (DETACH PARTITION 'Main')

Query id: 94112deb-b4cb-4d14-8575-dbef9a9ae577

Ok.

0 rows in set. Elapsed: 0.006 sec.
```

4. Проверяем:
```sql
SELECT
    partition,
    name,
    active
FROM system.parts
WHERE (`table` = 'menu_values_partitioned') AND (database = currentDatabase())

Query id: 1acf7a77-6717-4560-8b82-f2ee138630ee

   ┌─partition─┬─name───────────────────────────────────┬─active─┐
1. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_1_1_1 │      0 │
   └───────────┴────────────────────────────────────────┴────────┘

1 row in set. Elapsed: 0.003 sec.
```

5. Приатачим обратно:

```sql
ALTER TABLE menu_values_partitioned ATTACH PARTITION 'Main';
```

6. Проверяем:
```sql
SELECT
    partition,
    name,
    active
FROM system.parts
WHERE (`table` = 'menu_values_partitioned') AND (database = currentDatabase())

Query id: 952aa1f6-45ca-4f75-ae18-85bd6028a9cd

   ┌─partition─┬─name───────────────────────────────────┬─active─┐
1. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_1_1_1 │      0 │
2. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_2_2_0 │      1 │
   └───────────┴────────────────────────────────────────┴────────┘

2 rows in set. Elapsed: 0.003 sec. 
```
7. Дропаем партиции Main

```sql
ALTER TABLE menu_values_partitioned DROP PARTITION 'Main';
```

8. Проверяем:

```sql
SELECT
    partition,
    name,
    active
FROM system.parts
WHERE (`table` = 'menu_values_partitioned') AND (database = currentDatabase())

Query id: a41673f8-d319-4127-adc8-71a00aa9f96f

   ┌─partition─┬─name───────────────────────────────────┬─active─┐
1. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_1_1_1 │      0 │
2. │ Main      │ bb4089f103a121d3ee5e72f17a2529c7_2_2_1 │      0 │
   └───────────┴────────────────────────────────────────┴────────┘

2 rows in set. Elapsed: 0.003 sec.
```