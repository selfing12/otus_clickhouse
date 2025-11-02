# Домашнее задание 5 Джоины и агрегации.

## Цель:
Изучить синтаксис ClickHouse и базовые команды взаимодействия, освоить работу с джоинами, сложными типами данных и агрегацией, научиться создавать витрины данных.

## Задачи:

1. Создать базу и таблицы.
2. Загрузить тестовые данные с S3.
3. Выполнить все запросы по заданию.
4. Проверить корректность (первые 10 строк для каждого).
5. Сформировать отчёт.

### Создать БД и таблицы.

```sql

CREATE DATABASE imdb

Query id: bad9527d-f0e3-4cf9-ba3a-c9b6fbb7ca77

Ok.

0 rows in set. Elapsed: 0.004 sec.


CREATE TABLE imdb.actors
(
    `id` UInt32,
    `first_name` String,
    `last_name` String,
    `gender` FixedString(1)
)
ENGINE = MergeTree
ORDER BY (id, first_name, last_name, gender)

Query id: a97e483a-d5ef-48b9-8b29-88130d30aad6

Ok.

0 rows in set. Elapsed: 0.005 sec.


CREATE TABLE imdb.genres
(
    `movie_id` UInt32,
    `genre` String
)
ENGINE = MergeTree
ORDER BY (movie_id, genre)

Query id: 482ed6f4-f2ef-43af-b4cf-b74bad0e6ec4

Ok.

0 rows in set. Elapsed: 0.005 sec.


CREATE TABLE imdb.movies
(
    `id` UInt32,
    `name` String,
    `year` UInt32,
    `rank` Float32 DEFAULT 0
)
ENGINE = MergeTree
ORDER BY (id, name, year)

Query id: f4afc9ae-10f7-4bd8-96e0-2d7eabc1eb9a

Ok.

0 rows in set. Elapsed: 0.005 sec.


CREATE TABLE imdb.roles
(
    `actor_id` UInt32,
    `movie_id` UInt32,
    `role` String,
    `created_at` DateTime DEFAULT now()
)
ENGINE = MergeTree
ORDER BY (actor_id, movie_id)

Query id: cdd19b0c-dbd8-4051-a171-37696f6ca0a1

Ok.

0 rows in set. Elapsed: 0.006 sec.
```

### Вставить тестовые данные, используя функцию S3.
```sql
INSERT INTO imdb.actors SELECT *
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/imdb/imdb_ijs_actors.tsv.gz', 'TSVWithNames')

Query id: 0731684e-48c0-4e65-a953-7fa49878efc6

Ok.

0 rows in set. Elapsed: 1.362 sec. Processed 817.72 thousand rows, 26.71 MB (600.47 thousand rows/s., 19.61 MB/s.)
Peak memory usage: 48.36 MiB.


INSERT INTO imdb.genres SELECT *
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/imdb/imdb_ijs_movies_genres.tsv.gz', 'TSVWithNames')

Query id: a24662aa-1090-4c4c-96f7-d1f688851510

Ok.

0 rows in set. Elapsed: 0.347 sec. Processed 395.12 thousand rows, 6.62 MB (1.14 million rows/s., 19.07 MB/s.)
Peak memory usage: 15.03 MiB.


INSERT INTO imdb.movies SELECT *
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/imdb/imdb_ijs_movies.tsv.gz', 'TSVWithNames')

Query id: e6eb43c2-07e3-40d6-9b25-1845bdd00c76

Ok.

0 rows in set. Elapsed: 0.576 sec. Processed 388.27 thousand rows, 11.74 MB (674.34 thousand rows/s., 20.38 MB/s.)
Peak memory usage: 32.70 MiB.


INSERT INTO imdb.roles (actor_id, movie_id, role) SELECT
    actor_id,
    movie_id,
    role
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/imdb/imdb_ijs_roles.tsv.gz', 'TSVWithNames')

Query id: f0cd08f0-2f4b-4762-ada9-14b30b213c71

Ok.

0 rows in set. Elapsed: 2.400 sec. Processed 3.43 million rows, 87.51 MB (1.43 million rows/s., 36.46 MB/s.)
Peak memory usage: 117.27 MiB.
```
## Построение запросов, отвечающих на следующие задачи:
1. Найти жанры для каждого фильма.
```sql
SELECT
    m.name AS movie_name,
    g.genre
FROM imdb.movies AS m
INNER JOIN imdb.genres AS g ON m.id = g.movie_id
LIMIT 10

Query id: 304763a9-169d-4094-aad6-0af5d3f4b8ae

    ┌─movie_name────────────┬─genre────┐
 1. │ Patriot, The          │ War      │
 2. │ Patriot, The          │ Drama    │
 3. │ Patriot, The          │ Action   │
 4. │ Patriot, The (1998/I) │ Action   │
 5. │ Patriot, The (1998/I) │ Thriller │
 6. │ Patriot, The          │ Action   │
 7. │ Patriot, The          │ Drama    │
 8. │ Patriot, The          │ War      │
 9. │ Patriote, Le          │ Drama    │
10. │ Patrioten             │ Drama    │
    └───────────────────────┴──────────┘

10 rows in set. Elapsed: 0.100 sec. Processed 783.39 thousand rows, 19.95 MB (7.81 million rows/s., 198.93 MB/s.)
Peak memory usage: 72.71 MiB.
```
2. Запросить все фильмы, у которых нет жанра.
```sql
SELECT
    m.name,
    m.year
FROM imdb.movies AS m
LEFT JOIN imdb.genres AS g ON m.id = g.movie_id
WHERE g.movie_id IS NULL
LIMIT 10

Query id: 606e0647-6bb3-4a8d-9fb2-f001c56c2829

Ok.

0 rows in set. Elapsed: 0.013 sec.
```
3. Объединить каждую строку из таблицы `Фильмы` с каждой строкой из таблицы `Жанры`.
```sql

SELECT
    m.name AS movie_name,
    g.genre
FROM imdb.movies AS m
CROSS JOIN imdb.genres AS g
LIMIT 10

Query id: 2f8d8a45-84c2-4e20-95f3-b0bfaa8b974d

    ┌─movie_name─┬─genre───┐
 1. │ #28        │ Fantasy │
 2. │ #28        │ Horror  │
 3. │ #28        │ Comedy  │
 4. │ #28        │ Short   │
 5. │ #28        │ Drama   │
 6. │ #28        │ Drama   │
 7. │ #28        │ Comedy  │
 8. │ #28        │ Drama   │
 9. │ #28        │ Family  │
10. │ #28        │ Romance │
    └────────────┴─────────┘

10 rows in set. Elapsed: 0.024 sec. Processed 722.80 thousand rows, 15.12 MB (29.59 million rows/s., 619.20 MB/s.)
Peak memory usage: 24.58 MiB.
```
4. Найти жанры для каждого фильма, НЕ используя `INNER JOIN`.
```sql
SELECT
    m.name AS movie_name,
    g.genre
FROM imdb.movies AS m, imdb.genres AS g
WHERE m.id = g.movie_id
LIMIT 10

Query id: f11ae98a-de2c-44fc-aa1d-c3a54101305a

    ┌─movie_name───────────┬─genre───────┐
 1. │ Choshim              │ Drama       │
 2. │ Choshim              │ Romance     │
 3. │ Choti Si Dunya       │ Documentary │
 4. │ Chou                 │ Action      │
 5. │ Chou caillou hibou   │ Documentary │
 6. │ Chou de Cho-Juaa, El │ Animation   │
 7. │ Chou de Cho-Juaa, El │ Short       │
 8. │ Chou jiao Boluo      │ Crime       │
 9. │ Chou jue deng chang  │ Drama       │
10. │ Chou lian hu an      │ Action      │
    └──────────────────────┴─────────────┘

10 rows in set. Elapsed: 0.097 sec. Processed 783.39 thousand rows, 19.95 MB (8.08 million rows/s., 205.68 MB/s.)
Peak memory usage: 82.35 MiB.
```
5. Найти всех актеров и актрис, снявшихся в фильме в `N` году
```sql
SELECT
    a.first_name,
    a.last_name,
    m.name AS movie_name,
    m.year
FROM imdb.actors AS a
INNER JOIN imdb.roles AS r ON a.id = r.actor_id
INNER JOIN imdb.movies AS m ON r.movie_id = m.id
WHERE m.year = 2007
LIMIT 10

Query id: d7ec6643-2d61-46c5-b15a-f03602a2bf6c

   ┌─first_name─┬─last_name───┬─movie_name────────────────────────────────┬─year─┐
1. │ Tobey      │ Maguire     │ Spider-Man 3                              │ 2007 │
2. │ Ben        │ Kingsley    │ Tripoli                                   │ 2007 │
3. │ Reese      │ Witherspoon │ Rapunzel Unbraided                        │ 2007 │
4. │ Kristin    │ Chenoweth   │ Rapunzel Unbraided                        │ 2007 │
5. │ Jason      │ Isaacs      │ Harry Potter and the Order of the Phoenix │ 2007 │
   └────────────┴─────────────┴───────────────────────────────────────────┴──────┘

5 rows in set. Elapsed: 0.375 sec. Processed 4.64 million rows, 58.75 MB (12.36 million rows/s., 156.58 MB/s.)
Peak memory usage: 243.41 MiB.
```
6. Запросить все фильмы, у которых нет жанра, через `ANTI JOIN`
```sql
SELECT
    m.name,
    m.year
FROM imdb.movies AS m
ANTI LEFT JOIN imdb.genres AS g ON m.id = g.movie_id
LIMIT 10

Query id: eefb88fa-8f45-411f-8858-dc8fd1820739

    ┌─name───────────────────────────────┬─year─┐
 1. │ Express Train on a Railway Cutting │ 1898 │
 2. │ Expression, An                     │ 1988 │
 3. │ Expropiacin                        │ 1976 │
 4. │ Expropiacin, La                    │ 1972 │
 5. │ Expulsion from Hell                │ 1988 │
 6. │ Expsito, El                        │ 1920 │
 7. │ Exquisite Corpses                  │ 1989 │
 8. │ Exquisite Excesses                 │ 1999 │
 9. │ Exquisite Feet                     │ 2002 │
10. │ Extase                             │ 1932 │
    └────────────────────────────────────┴──────┘

10 rows in set. Elapsed: 0.059 sec. Processed 783.39 thousand rows, 15.42 MB (13.24 million rows/s., 260.71 MB/s.)
Peak memory usage: 34.24 MiB.
```

### Источники и справочные материалы:
https://clickhouse.com/docs/en/integrations/dbt
https://clickhouse.com/blog/clickhouse-fully-supports-joins-part1 