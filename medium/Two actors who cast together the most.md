## Two actors who cast together the most

Ссылка на задачу: `https://www.codewars.com/kata/5818bde9559ff58bd90004a2/train/sql`

> **Мои мысли**
>
>Очень простая на первый взгляд задача, однако чтобы ее решить пришлось поломать голову. Мой вариант решения оказался не типичным для такой задачи, но мне он очень понравился.


### Задание
На основе представленной ниже схемы найдите двух актёров, которые чаще всего снимались вместе, и перечислите названия только тех фильмов, в которых они играли. Отсортируйте результат по названию фильма в алфавитном порядке.


### Таблицы

<br>

**film_actor**

Column      | Type                        | Modifiers
------------|-----------------------------|----------
actor_id    | smallint                    | not null
film_id     | smallint                    | not null
...

<br>

**actor**

Column      | Type                        | Modifiers
------------|-----------------------------|----------
actor_id    | integer                     | not null 
first_name  | character varying(45)       | not null
last_name   | character varying(45)       | not null
...

<br>

**film**
 Column     | Type                        | Modifiers
------------|-----------------------------|----------
film_id     | integer                     | not null
title       | character varying(255)      | not null
...

<br>

**The desired output:**

first_actor | second_actor | title
------------|--------------|--------------------
John Doe    | Jane Doe     | The Best Movie Ever
...

`first_actor` - Полное имя (First name + Last name разделенные пробелом)

`second_actor` - Полное имя (First name + Last name разделенные пробелом)

`title` - Название фильма

**Примечание:** actor_id первого актора (first_actor) должен быть меньше actor_id второго актора (second_actor)



## Мое решение
> Задание прописано плохо, поэтому будет реализованно решение, полагая пара актеров должна быть только одна.

Я использую нестандартное применение LIMIT в последней строке и это работает, но при желании код можно было бы переписать в более простой вид используя дополнительный подзапрос и функцию RANK()



~~~~sql
WITH cte as (
SELECT * FROM film_actor
JOIN actor USING(actor_id)
JOIN film USING(film_id)
),

counted_data as (
SELECT
t1.first_name || ' ' || t1.last_name as first_actor,
t2.first_name || ' ' || t2.last_name as second_actor,
t1.title,
COUNT(*) OVER (partition by t1.actor_id, t2.actor_id) as cnt
FROM cte t1, cte t2
WHERE t1.actor_id < t2.actor_id  and t1.film_id = t2.film_id
ORDER BY cnt DESC, title
)

SELECT first_actor, second_actor, title FROM counted_data
LIMIT (SELECT cnt FROM counted_data LIMIT 1)
~~~~

